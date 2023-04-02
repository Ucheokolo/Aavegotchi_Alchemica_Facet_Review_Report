# Okolo Uchenna

## Contract Review of Aavegotchi Alchemica Faucet.

### Contents

- Contract Overview
- Faucet Review
  - Overview
  - Code-base Review
- Summary

### Contract Overview

[Aavegotchi](https://www.aavegotchi.com/) is a Defi-enabled crypto collectibles game developed by Singapore-based Pixelcraft studios.
The platform allows players to stake NFTs with interest generating token and interact with its metaverse.
It is a unique combination of Decentralized Finance (DeFi) and NFTs.
Aavegotchis are pixelated ghosts living on the Ethereum blockchain, backed by the ERC-721 standard.
Their value is determined by rarity level, which is calculated via multiple factors, such as base traits,
amount of staked collateral/aTokens, and equipped wearables.
The contract was developed using EIP2535 (i.e Diamond Standard).

### Faucet Review

#### Overview

**Introduction** <br>
The Alcemica Faucet is the code base implementation of the [Gotchus Alchemica](https://wiki.aavegotchi.com/en/gotchiverse#gotchus-alchemica) use case in tandem with Realm Parcels.
Gotchus Alchemica are the four elements of the Gotchiverse; FUD, FOMO, ALPHA, and KEK.
They are fair-launch ERC20 tokens that are used to craft Installation NFTs within the Gotchiverse.

**How To Earn Alchemica** <br>
_Players can earn Alchemica in three distinct ways_:

1. farming from their REALM parcels.
2. **Alchemical Channeling** which comes as daily stipends from the gotchiverse as daily stipends from Aavegotchis.
3. **Communal Channeling** which comes as result of lodge installations built upon realm parcels.
4. Gotchus Alchemica can also be exchanged for GHST (i.e Aavegotchi eco-governance token) using our native DEX, the Gotchus Alchemica exchange (GAX).

#### Code-base Review

**NB** _Recall that facets in EIP2535 hold no state variables, hence the reason for a stateless smart contract_ (i.e Alchemica Facet).<br>
_Aavegotchi has 3 main Realms(Human Realm, Ether Realm; where smart contracts twinkle and dark forests lurk, and Gotchiverse Realm)_ <br>
<q>
_When a yield farmer in the Ether Realm is liquidated, its spirit journeys to the Gotchiverse, where it reincarnates as an Aavegotchi_
</q> ~ [Gotchiverse](https://wiki.aavegotchi.com/en/gotchiverse)

The Gotchiverse Realm is where most of the game play takes place.

**The Alchemica Faucet utilized the following dependencies in its contract**.

1. "AppStorage"; which refrences Diamond state Variables.
2. "LibAlchemica"; Which contains the function logics for the Alchemica Facet.
3. "RealmFacet"; Which is the code base implementation of Realm Parcel in [Gotchiverse](https://wiki.aavegotchi.com/en/gotchiverse)
4. "LibMeta.sol";
5. "VRFCoordinatorV2Interface";
6. "LibSignature";
7. "IERC20Extended";

**Functions Review** <br>
The Gotchiverse houses Districts that comprises of Realm parcels. Users can obtain this Realm parcels. This forms the basis for
the first function in Alchemica Facet.

This function and subsequent ones references _"s.parcels"_.<br>
Futher reviewing the contract, parcels is a variable of mapping to struct _"Parcel"_. It initializes the Parcel struct from AppStorage.

1. function isSurveying
<pre>function isSurveying(uint256 _realmId) external view returns (bool) {
    return s.parcels[_realmId].surveying;
  }</pre>

- This fuction returns a boolean value(True or False) to indicates an onwer's realm parcel is being surveyed by owner or not.
- It checks this by looking into parcels to get the value for key "surveying".

2. function startSurveying
<pre>

function startSurveying(uint256 \_realmId) external onlyParcelOwner(\_realmId) gameActive {
//current round and surveying round both begin at 0.
//after calling VRF, currentRound increases
require(s.parcels[_realmId].currentRound <= s.surveyingRound, "AlchemicaFacet: Round not released");
require(s.parcels[_realmId].altarId > 0, "AlchemicaFacet: Must equip Altar");
require(!s.parcels[_realmId].surveying, "AlchemicaFacet: Parcel already surveying");
s.parcels[_realmId].surveying = true;
// do we need to cancel the listing?
drawRandomNumbers(\_realmId, s.parcels[_realmId].currentRound);

    emit StartSurveying(_realmId, s.parcels[_realmId].currentRound);

}</pre>

- Allow the owner of a parcel to start surveying his parcel
- Will throw if a surveying round has not started
- realmId Identifier of the parcel to survey
