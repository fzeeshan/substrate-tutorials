# Check out the correct repo using POE Directory
In root director, follow instructions inside POE tutorial to ensure you have the correct repo and tools installed.
This tutorial will work only on correct version of Substrate.

# add new module to pay transaction fees to block author.
In addition to this functionality, block rewards could also be added to pay a certain amount of coins to block author.
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
### Edit lib.rs
open runtime/src/lib.rs
make the following changes
add 
mod impls;

```sh
//------------------------------
pub struct LinearWeightToFee<C>(sp_std::marker::PhantomData<C>);
impl<C> frame_support::weights::WeightToFeePolynomial for LinearWeightToFee<C>
where
	C: frame_support::pallet_prelude::Get<Balance>,
{
	type Balance = Balance;

	fn polynomial() -> frame_support::weights::WeightToFeeCoefficients<Self::Balance> {
		let coefficient = frame_support::weights::WeightToFeeCoefficient {
			coeff_integer: C::get(),
			coeff_frac: Perbill::zero(),
			negative: false,
			degree: 1,
		};

		// Return a smallvec of coefficients. Order does not need to match degrees
		// because each coefficient has an explicit degree annotation.
		smallvec!(coefficient)
	}
}

pub struct QuadraticWeightToFee;
impl frame_support::weights::WeightToFeePolynomial for QuadraticWeightToFee {
	type Balance = Balance;
	fn polynomial() -> frame_support::weights::WeightToFeeCoefficients<Self::Balance> {
		let linear = frame_support::weights::WeightToFeeCoefficient {
			coeff_integer: 2,
			coeff_frac: Perbill::from_percent(40),
			negative: true,
			degree: 1,
		};
		let quadratic = frame_support::weights::WeightToFeeCoefficient {
			coeff_integer: 3,
			coeff_frac: Perbill::zero(),
			negative: false,
			degree: 2,
		};

		// Return a smallvec of coefficients. Order does not need to match degrees
		// because each coefficient has an explicit degree annotation. In fact, any
		// negative coefficients should be saved for last regardless of their degree
		// because large negative coefficients will likely cause saturation (to zero)
		// if they happen early on.
		smallvec![quadratic, linear]
	}
}

parameter_types! {
	// Used with LinearWeightToFee conversion. Leaving this constant in tact when using other
	// conversion techniques is harmless.
	pub const FeeWeightRatio: u128 = 1_000;

	// Establish the byte-fee. It is used in all configurations.
	pub const TransactionByteFee: u128 = 1;
}
//=====================
```
update impl pallet_transaction_payment::Config for Runtime
change the following line type OnChargeTransaction = CurrencyAdapter<Balances, ()>;
to
type OnChargeTransaction = CurrencyAdapter<Balances, crate::impls::DealWithFees>;

also change 	//type WeightToFee = IdentityFee<Balance>;
to
type WeightToFee = LinearWeightToFee<FeeWeightRatio>;

add the following code:
```sh
// added for fee payment to block author.

pub struct AuraAccountAdapter;

impl frame_support::traits::FindAuthor<AccountId> for AuraAccountAdapter {
	fn find_author<'a, I>(digests: I) -> Option<AccountId>
		where I: 'a + IntoIterator<Item=(frame_support::ConsensusEngineId, &'a [u8])>
	{
		pallet_aura::AuraAuthorId::<Runtime>::find_author(digests).and_then(|k| {
			AccountId::try_from(k.as_ref()).ok()
		})
	}
}

impl pallet_authorship::Config for Runtime {
	type FindAuthor = AuraAccountAdapter;
	//polkadot-v0.9.23 
	//type UncleGenerations = ();
	//polkadot-v0.9.23 
	//type FilterUncle = ();
	type EventHandler =  ();
}
// End here -> Fees for block author
```
```
update 
  construct_runtime!(
	pub struct Runtime {
		System: frame_system,
		Timestamp: pallet_timestamp,
		Aura: pallet_aura,
		Grandpa: pallet_grandpa,
		Balances: pallet_balances,
		TransactionPayment: pallet_transaction_payment,
		Sudo: pallet_sudo,
		// Include the custom logic from the pallet-template in the runtime.
		TemplateModule: pallet_template,
```
  add the following line
```sh
Authorship:pallet_authorship,
```

Check that the new dependencies resolve correctly by running the following command:
```sh
cargo +nightly-2023-06-15 check -p node-template-runtime --release
```
```sh
cargo +nightly-2023-06-15 build --release
```
## Launch one or more nodes
