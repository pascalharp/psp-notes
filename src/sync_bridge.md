# Sync Bridge
Sync Bridge is a small little python script to synchronize two running gdb instances. It is used to compare the behavior of PSPEmu and the Qemu port. It currently only compares the PC and stops once they differ.

## python script
This will/should be moved to it's own repository at some point.
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# default values
HOST = 'localhost'
PORT = 8123
TIMEOUT = 5

import socket
import gdb

class Plugin(gdb.Command):

    def __init__(self):
        print("initializing sync bridge")
        gdb.Command.__init__(self, "sb_init", gdb.COMMAND_OBSCURE, gdb.COMPLETE_NONE)
        self.host = None
        self.port = None

    def invoke(self, arg, from_tty):
        # TODO parse args for custom host and port
        self.host = HOST
        self.port = PORT

        # register other commands
        cmds = [BridgeFollow(self), BridgeLead(self)]

        print("Registered {} commands".format(len(cmds)))

class BridgeFollow(gdb.Command):

    def __init__(self, plug):
        self.plug = plug
        gdb.Command.__init__(self, "sb_follow", gdb.COMMAND_OBSCURE, gdb.COMPLETE_NONE)
        self.sock = None

    def invoke(self, arg, from_tty):

        print("Connecting to {} on port {} as follower".format(self.plug.host, self.plug.port))
        try:
            self.sock = socket.create_connection((self.plug.host, self.plug.port), TIMEOUT)
        except socket.error as err:
            if self.sock:
                self.sock.close()
                self.sock = None
            print("Connection error: {}".format(err))
            return None

        try:
            leader_pc = self.sock.recv(1024).decode()
            pc = str(gdb.parse_and_eval("$pc"))
            self.sock.send(leader_pc.encode())
            print("Own pc {}, leader pc {}".format(pc, leader_pc))

            while leader_pc == pc:
                print("PONG")
                gdb.execute("si")
                leader_pc = self.sock.recv(1024).decode()
                pc = str(gdb.parse_and_eval("$pc"))
                self.sock.send(pc.encode())

            print("Leader pc {} did not match own pc {}".format(leader_pc, pc))

        except Exception as err:
            print("Error while bridge sync: {}".format(err))

        self.sock.close()
        self.sock = None

class BridgeLead(gdb.Command):

    def __init__(self, plug):
        self.plug = plug
        gdb.Command.__init__(self, "sb_lead", gdb.COMMAND_OBSCURE, gdb.COMPLETE_NONE)
        self.sock = None
        self.client_sock = None
        self.client_addr = None

    def invoke(self, arg, from_tty):

        print("Connecting to {} on port {} as leader".format(self.plug.host, self.plug.port))
        try:
            self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.sock.bind((self.plug.host, self.plug.port))
            self.sock.listen(1)
        except socket.error as err:
            if self.sock:
                self.sock.close()
                self.sock = None
            print("Connection error: {}".format(err))
            return None

        print("Waiting for follower...")
        (self.client_sock, self.client_addr) = self.sock.accept()
        print("Client connected")

        try:
            pc = str(gdb.parse_and_eval("$pc"))
            self.client_sock.send(pc.encode())
            follow_pc = self.client_sock.recv(1024).decode()
            print("Own pc {}, follow pc {}".format(pc, follow_pc))
            while follow_pc == pc:
                gdb.execute("si")
                pc = str(gdb.parse_and_eval("$pc"))
                self.client_sock.send(pc.encode())
                follow_pc = self.client_sock.recv(1024).decode()

            print("Follow pc {} did not match own pc {}".format(follow_pc, pc))

        except Exception as err:
            print("Error while bridge sync: {}".format(err))

        self.sock.close()
        self.sock = None

if __name__ == "__main__":
    try:
        id(SYNC_BRIDGE)
        print("Plugin already loaded")
    except:
        SYNC_BRIDGE = Plugin()
```

## Usage

First start the Qemu and the PSPEmu emulation with gdb. Different ports are required. Here PSPEmu will be on port `1235` and Qemu will be on the default port `1234`.

Replace `PSP_ROM_BL` with the full path to the on-chip bootloader and `PSP_UEFI_IMAGE` with the full path to the uefi image.

Now start two `arm-none-eabi-gdb` instances and connect one to the Qemu emulation and one to the PSPEmu emulation. Source the sync\_bridge plugin and initialize it. Now first start one of them first as the leader with `sb_leader` and afterwords the other one as the follower `sb_follow`. Make sure they start at the same address. Check the scripts below on how to automate this process.

### Qemu emulation
```bash
#!/bin/env bash
path/to/qemu/build/qemu-system-arm \
	--singlestep \
	--machine amd-psp_zen \
	--display none \
	-device loader,file=$PSP_ROM_BL,addr=0xffff0000,force-raw=on \
	-bios $PSP_UEFI_IMAGE \
	-serial stdio \
	-d unimp,guest_errors
	-S -s
```

### PSPEmu emulation
```bash
#!/bin/env bash
path/to/PSPEmu/build/PSPEmu \
    --emulation-mode on-chip-bl \
    --flash-rom $PSP_UEFI_IMAGE \
    --on-chip-bl $PSP_ROM_BL \
    --timer-real-time \
    --trace-log ./log \
    --intercept-svc-6 \
    --trace-svcs \
    --dbg 1235
```

### GDB qemu
```bash
#!/bin/env bash
arm-none-eabi-gdb \
	-ex 'source ~/.gdb/sync.py' \         # Only if combined with ret-sync
	-ex 'pi reset_architecture("arm")' \
	-ex 'source ~/.gdb/sync_bridge.py' \
	-ex 'sb_init' \
	-ex 'gef-remote localhost:1234' \     # If not using gef replace with: target remote localhost:1234
	-ex 'break *0xffff005c' \
	-ex 'continue' \
	-ex 'sync' \                          # Only if combined with ret-sync
	-ex 'sb_lead'
```

### GDB PSPEmu
```bash
#!/bin/env bash
arm-none-eabi-gdb \
	-ex 'pi reset_architecture("arm")' \
	-ex 'source ~/.gdb/sync_bridge.py' \
	-ex 'sb_init' \
	-ex 'gef-remote localhost:1235' \     # If not using gef replace with: target remote localhost:1235
	-ex 'break *0xffff005c' \
	-ex 'continue' \
	-ex 'sb_follow'
```
