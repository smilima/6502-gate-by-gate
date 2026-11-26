# Chapter 1: build a circuit and send it to the Tang Nano 20K.
#   make gate     build the one-gate circuit
#   make gates    build the six gates, one per LED
#   make blink    build the blinking LED
#   make flash-gate / flash-gates / flash-blink    send it to the board
#   make keep-gate    write it into the board's permanent memory instead
BUILD  ?= build
DEVICE  = GW2AR-LV18QN88C8/I7
FAMILY  = GW2A-18C
FREQ    = 27

gate gates blink: %: $(BUILD)/%.fs

$(BUILD)/%.json: %.v
	mkdir -p $(BUILD)
	yosys -q -p "read_verilog $<; synth_gowin -top top -json $@"

$(BUILD)/%_pnr.json: $(BUILD)/%.json %.cst
	nextpnr-himbaechel --json $< --write $@ --device $(DEVICE) \
		--vopt family=$(FAMILY) --vopt cst=$*.cst --freq $(FREQ)

$(BUILD)/%.fs: $(BUILD)/%_pnr.json
	gowin_pack -d $(FAMILY) -o $@ $<

flash-%: $(BUILD)/%.fs
	openFPGALoader -b tangnano20k $<

# The same, into the board's permanent memory: the circuit will still be
# there when the board is unplugged and plugged in again.
keep-%: $(BUILD)/%.fs
	openFPGALoader -b tangnano20k -f $<

clean:
	rm -rf $(BUILD)

.PHONY: gate gates blink clean
.PRECIOUS: $(BUILD)/%.json $(BUILD)/%_pnr.json

# Connect the board to WSL. Needed after every unplug, reboot or wsl --shutdown.
# The board is found by its USB ID (VID:PID), so any USB port works.
USBIPD = "/mnt/c/Program Files/usbipd-win/usbipd.exe"
HWID ?= 0403:6010
ATTACH_TIMEOUT ?= 4
attach:
	@if lsusb | grep -q $(HWID); then \
		echo "Board is already attached to WSL."; \
	else \
		timeout $(ATTACH_TIMEOUT) $(USBIPD) attach --wsl --hardware-id $(HWID); rc=$$?; \
		if [ $$rc -eq 124 ]; then \
			/mnt/c/Windows/System32/wsl.exe -u root -e pkill -f '^\./usbip attach'; \
			echo "Attach timed out after $(ATTACH_TIMEOUT) s. A Windows program may be holding the board."; \
			exit 1; \
		fi; \
		[ $$rc -eq 0 ] || exit $$rc; \
		sleep 1; \
	fi
