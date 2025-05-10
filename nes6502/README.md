# NES 6502

Test data for the 6502-derived CPU core contained within the RP2A03 microprocessor used by the NES.

This version of the 6502 disabled the binary-coded decimal mode.

For tests of the original 6502, refer to [6502](../6502/) instead.

# Guide

Within each `.json` file, there is a JSON array of 10,000 test scenarios.

Each file corresponds to each of the instruction opcodes. (e.g. `a9.json` stores all of the tests for the LDA immediate instruction)

Each test contains a `name` string, which can be ignored, and three major sections:
* `initial` - the initial state of the processor
* `final` - the expected final state of the processor
* `cycles` - a list of expected bus operations for each cycle of the instruction
  * each cycle is in the form of `[address, value, type]` where `type` is either `"read"` or `"write"`

## Memory

All tests assume the entire 64KiB memory space is RAM. Since this is different from the memory layout of the NES, you _must_ write a separate memory map for your emulator in order to perform these tests.

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
	"name": "b1 71 8b",
	"initial": {
		"pc": 9023,
		"s": 240,
		"a": 47,
		"x": 162,
		"y": 170,
		"p": 170,
		"ram": [
			[9023, 177],
			[9024, 113],
			[9025, 139],
			[113, 169],
			[114, 89],
			[22867, 214],
			[23123, 37]
		]
	},
	"final": {
		"pc": 9025,
		"s": 240,
		"a": 37,
		"x": 162,
		"y": 170,
		"p": 40,
		"ram": [
			[113, 169],
			[114, 89],
			[9023, 177],
			[9024, 113],
			[9025, 139],
			[22867, 214],
			[23123, 37]
		]
	},
	"cycles": [
		[9023, 177, "read"],
		[9024, 113, "read"],
		[113, 169, "read"],
		[114, 89, "read"],
		[22867, 214, "read"],
		[23123, 37, "read"]
	]
}
```
