# Rockwell 65C02

Test data for the Rockwell 65C02 microprocessor.

This version of the 65C02 contains all of the added instructions of the Rockwell Synertek: `P[H/L][X/Y]`, `STZ`, `TRB`, `TSB`, `BBR`, `BBS`, `RMB`, `SMB`, and the (zp) addressing mode.

# Guide

Within each `.json` file, there is a JSON array of 10,000 test scenarios.

Each file corresponds to each of the instruction opcodes. (e.g. `a9.json` stores all of the tests for the LDA immediate instruction)

Each test contains a `name` string, which can be ignored, and three major sections:
* `initial` - the initial state of the processor
* `final` - the expected final state of the processor
* `cycles` - a list of expected bus operations for each cycle of the instruction
  * each cycle is in the form of `[address, value, type]` where `type` is either `"read"` or `"write"`

## Memory

All tests assume the entire 64KiB memory space is RAM. In order to perform these tests, you may need to write a separate memory map for your emulator.

The `ram` elements inside the `initial` and `final` sections of the test, are in the form of `[address, value]`.

Any memory address not included in a test's `ram` lists must not be accessed during that test. However, there may be memory addresses in the `ram` lists that are never accessed during the test.

## Test Procedure

First use a JSON parser to parse one of the `.json` files into an array/list of individual tests.

Then for each test:
1. Initialize your emulator's processor and memory with the `initial` state data provided by the test.
2. Step your emulator forward until the processor completes **one instruction** worth of cycles.
3. (Optional) After each cycle, compare your processor's last bus operation against the corresponding cycle from the `cycles` list.
4. Once the instruction is complete, compare the test's expected `final` state with your emulator's state. If everything matches, you've passed the test!

If you aren't emulating individual cycles (skipped step #3), you can still count how many elements are contained in the `cycles` list and compare with the number of cycles your emulator should have taken to perform the instruction.

If you do perform step #3, also ensure only **one** bus operation occurs per cycle.

## Example

Below is an example of a single test:
```JSON
{
	"name": "b1 28 b5",
	"initial": {
		"pc": 59082,
		"s": 39,
		"a": 57,
		"x": 33,
		"y": 174,
		"p": 96,
		"ram": [
			[59082, 177],
			[59083, 40],
			[59084, 181],
			[40, 160],
			[41, 233],
			[59982, 119]
		]
	},
	"final": {
		"pc": 59084,
		"s": 39,
		"a": 119,
		"x": 33,
		"y": 174,
		"p": 96,
		"ram": [
			[40, 160],
			[41, 233],
			[59082, 177],
			[59083, 40],
			[59084, 181],
			[59982, 119]
		]
	},
	"cycles": [
		[59082, 177, "read"],
		[59083, 40, "read"],
		[40, 160, "read"],
		[41, 233, "read"],
		[59083, 40, "read"],
		[59982, 119, "read"]
	]
}
```
