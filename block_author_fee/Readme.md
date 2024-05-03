# Work In Progress
Will be adding tutorial on how to add block authoring reward. This is different from block reward, will eventually implment block reward also, for now, this will show how to pay a portion of fee to the validator that created the block. This works much like Bitcoin but in a POA/POS model.

## How to pay fee to block author as a reward.
check out project in poe directory.
open runtime/Cargo.toml
Add the following under 
[dependencies]
```sh
smallvec = "1.4"
pallet-authorship = { default-features = false, version = "4.0.0-dev", git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v1.0.0" }
```
under 
[features]
default = ["std"]
std = [
add the following 
```sh
"pallet-authorship/std",
```
Save and close
### add impls.rs
inside runtime/src create a new file called impls.rs
add the following within impls.rs
```sh
use crate::{Authorship, Balances};
use frame_support::traits::{Imbalance, OnUnbalanced};
use crate::sp_api_hidden_includes_construct_runtime::hidden_include::traits::Currency;
use crate::AccountId;

type NegativeImbalance = <Balances as Currency<AccountId>>::NegativeImbalance;

pub struct Author;
impl OnUnbalanced<NegativeImbalance> for Author {
	fn on_nonzero_unbalanced(amount: NegativeImbalance) {
		if let Some(author) = Authorship::author() {
			Balances::resolve_creating(&author, amount);
		}
	}
}

pub struct DealWithFees;
impl OnUnbalanced<NegativeImbalance> for DealWithFees {
	fn on_unbalanceds<B>(mut fees_then_tips: impl Iterator<Item = NegativeImbalance>) {
		if let Some(fees) = fees_then_tips.next() {
			let mut split = fees.ration(0, 100);
			if let Some(tips) = fees_then_tips.next() {
				// for tips, if any, 80% to treasury, 20% to block author (though this can be anything)
				tips.ration_merge_into(0, 100, &mut split);
			}
			//Treasury::on_unbalanced(split.0);
			Author::on_unbalanced(split.1);
		}
	}
} 
```
