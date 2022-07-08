# Squads Multisig Program Library
The program facilitates signing and executing transactions on behalf of a multisig, and is currently in Alpha. The program is written in [Anchor](https://github.com/coral-xyz/anchor), with instructions and accounts that can be easily deserialized by the program IDL.
## Accounts
There are 3 types of accounts in the program
* Multisig ([Ms](https://github.com/squads-dapp/squads-mpl/blob/main/programs/squads-mpl/src/state/ms.rs#L6]))
* Transaction ([MsTransaction](https://github.com/squads-dapp/squads-mpl/blob/main/programs/squads-mpl/src/state/ms.rs#L94))
* Instruction ([MsInstruction](https://github.com/squads-dapp/squads-mpl/blob/main/programs/squads-mpl/src/state/ms.rs#L235))

## Instructions
Instructions can be categorized as such:
* Internal Instructions, which are the squads-mpl instructions invoked directly
* External/Arbitrary Instructions, which can be attached to transactions that will ultimately be executed by the multisig
### Internal Instructions
Internal instructions that primarily affect the Ms account:
* Create
* Add Member
* Remove Member
* Change Threshold
* Add Member & Change Threshold
* Remove Member & Change Threshold

Internal instructions related to handling MsTransactions:
* Create
* Attach External/Abitrary Instruction
* Sign off / Activate
* Approve
* Reject
* Cancel
* Execute

## Authorities
Each created and executed MsTransaction does so on behalf of an authority. Authorities are derived by a u32, and saved in the MsTransaction account when created (by passing in the `authority_index` argument). The Authority with an index of 0 is reserved for MsTransactions that affect the multisig directly (add member, change threshold, etc). Other authority indexes are agnostic and represent the underlying account/PDA that will be signed for during execution. For example, a multisig can use `authority_index 1` for a vault, `authority_index 2` for a secondary vault, and `authority_index 3` for a program upgrade authority. It is up to the end user to decide how to leverage these and to make sure that the `authority_index` in the created MsTransaction matches the necessary accounts specified in the attached instructions.

## Execute a MsTransaction
In order to execute a MsTransaction, in addition to the accounts specified in the IDL, the user/key invoking the execute must also pass in a list of accounts that reference the MsInstructions in this format (example for 2 instructions):

First MsInstruction (`instruction_index of 1`)
* The PDA of the MsInstruction
* The program_id that will be invoked by the MsInstruction
* A list of all other accounts referenced by the attached MsInstruction
  
Second MsInstruction (`instruction_index of 2`)
* The PDA of the MsInstruction
* The program_id that will be invoked by the MsInstruction
* A list of all other accounts referenced by the attached MsInstruction

The accounts needed for execution can be derived by the MsTransaction account itself, as the MsTransaction account contains an instruction_index, which when attaching an MsInstruction needs to be incremented sequentially. To execute, first you can fetch the MsTransaction account, and then derive all MsInstruction accounts by working backwards from the instruction_index in the MsTransaction and derive the MsInstruction PDAs, fetch the MsInstruction accounts, and then format the ExecuteInstruction for the multisig as explained above. See how this can be accomplished you can [take a look here at one of the test helper functions](https://github.com/squads-dapp/squads-mpl/blob/main/helpers/transactions.ts#L29).