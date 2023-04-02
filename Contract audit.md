# Web3Bridge Assigment

## Author: Okolo Uchenna

## Title: Contract Review of Aavegotchi Alchemica Faucet.

<hr>

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

1. **farming** from their REALM parcels.
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
<hr>

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
- It requires that requires that currentRound <= s.surveyingRound ("s" is referencing the diamond storage).<br>This is because after each round of Survey, surveyingRound is increased to value of currentRound.
- it requires owner to install atleast an Altar(Initially used for Alchemical Channeling)
- function Will throw an error message if a surveying round has not started
- The drawRandom number function is inherited from the chainLink VRF and is used to generate the number of materials in the
  Real parcel.
  <hr>

3. function drawRandomNumber
<pre>
function drawRandomNumbers(uint256 _realmId, uint256 _surveyingRound) internal {
  // Will revert if subscription is not set and funded.
  uint256 requestId = VRFCoordinatorV2Interface(s.vrfCoordinator).requestRandomWords(
    s.requestConfig.keyHash,
    s.requestConfig.subId,
    s.requestConfig.requestConfirmations,
    s.requestConfig.callbackGasLimit,
    s.requestConfig.numWords
  );
  s.vrfRequestIdToTokenId[requestId] = _realmId;
  s.vrfRequestIdToSurveyingRound[requestId] = _surveyingRound;
}
</pre>

- This function utilizes the chainlink VRF to generate the random number used in the ststartSurveying function.
<hr>

4. function getAlchemicaAddresses()
<pre>
function getAlchemicaAddresses() external view returns (address[4] memory) {
    return s.alchemicaAddresses;
  }
</pre>

- This basically returns a fixed array of addresses for the 4 Alchemica elements(ERC20 token).
**NB**: Recall there are potentially 4 Alchemicas (FOMO, KEK, ALPHA, FUD). The Address required is one of those as revealed in parcel.
<hr>

5. function getTotalAlchemicas()
<pre>
function getTotalAlchemicas() external view returns (uint256[4][5] memory) {
return s.totalAlchemicas;
}
</pre>

- returns A two dimensional array that returns an Alchemica (FOMO, KEK, ALPHA, FUD) with is corresponding value.
<hr>

6. function getRealmAlchemica
<pre>
function getRealmAlchemica(uint256 \_realmId) external view returns (uint256[4] memory) {
   return s.parcels[_realmId].alchemicaRemaining;
   }
</pre>

- This function queries and returns the amount of Alchemica remaining in a real parcel.
- Its takes "\_realmID " (which is basically the parcel Identifier) as an argument and returns details about remaining Alchemicas in the parcel.
- This is also an array of integers.
<hr>

7. function getParcelCurrentRound
<pre>
function getParcelCurrentRound(uint256 _realmId) external view returns (uint256) {
    return s.parcels[_realmId].currentRound;
  }
</pre>

**NB**: Recall startSurveying function. remember that after each round of Survey, surveyingRound is increased to value of currentRound.

- CurrentRound here refers to current survey round and they start at zero and increment by one after each round of survey.
<hr>

8. function progressSurveyingRound()
<pre>
function progressSurveyingRound() external onlyOwner {
    s.surveyingRound++;
    emit SurveyingRoundProgressed(s.surveyingRound);
  }
</pre>

- This further solidifies the startSurveying and getParcelCurrentRound functions. the diamond Owner increments this at the end of each round of survey.
<hr>

9. function getRoundAlchemica
<pre>
function getRoundAlchemica(uint256 _realmId, uint256 _roundId) external view returns (uint256[] memory) {
    return s.parcels[_realmId].roundAlchemica[_roundId];
  }
</pre>

**NB**: Depending on Realm parcel size (i.e Humble, Reasonable, Spacious vertical, Spacious horizontal and Partner) each parcel gets an alchemica boost. This simply means more Alchemica in your parcel.

- This function takes realmID and roundID as arguments to return an array of all Alchemicas gotten from parcel, this includes boosts.
<hr>

10. function getRoundBaseAlchemica
<pre>
function getRoundBaseAlchemica(uint256 _realmId, uint256 _roundId) external view returns (uint256[] memory) {
    return s.parcels[_realmId].roundBaseAlchemica[_roundId];
  }
</pre>

- This function does retrieves the same details as the getRoundAlchemica function with the exception of boosts.
<hr>

11. function setVars
<pre>
function setVars() external onlyOwner {
    
    s.boostMultipliers = _boostMultipliers;
    s.greatPortalCapacity = _greatPortalCapacity;
    s.installationsDiamond = _installationsDiamond;
    s.vrfCoordinator = _vrfCoordinator;
    s.linkAddress = _linkAddress;
    s.alchemicaAddresses = _alchemicaAddresses;
    s.backendPubKey = _backendPubKey;
    s.gameManager = _gameManager;
    s.gltrAddress = _gltrAddress;
    s.tileDiamond = _tileDiamond;
    s.aavegotchiDiamond = _aavegotchiDiamond;
  }
</pre>

**NB**: The Length/contents of this function was removed for readability. It still contains the significant informations.

- This allows diamond owner to set the foolowing state variables.
- alchemicas: A nested array containing the amount of alchemicas available
- boostMultipliers: The boost multiplers applied to each parcel
- greatPortalCapacity: The individual alchemica capacity of the great portal(This where Aavegotchis are summoned)
- installationsDiamond: The installations diamond address
- vrfCoordinator: The chainlink vrfCoordinator address
- linkAddress: The link token address
- alchemicaAddresses: The four alchemica token addresses (FOMO, FUD, KEK, ALPHA)
- backendPubKey: The Realm(gotchiverse) backend public key
- gameManager: The address of the game manager
<hr>

12. function setTotalAlchemicas
<pre>
function setTotalAlchemicas(uint256[4][5] calldata _totalAlchemicas) external onlyOwner {
    for (uint256 i; i < _totalAlchemicas.length; i++) {
      for (uint256 j; j < _totalAlchemicas[i].length; j++) {
        s.totalAlchemicas[i][j] = _totalAlchemicas[i][j];
      }
    }
  }
</pre>

- This fuction takes a nested array of Uint as argument and loops through the array of Alchemicas and assigns the value for each Alchemica.
<hr>

13. function getAvailableAlchemica
<pre>
  function getAvailableAlchemica(uint256 _realmId) public view returns (uint256[4] memory _availableAlchemica) {
    for (uint256 i; i < 4; i++) {
      _availableAlchemica[i] = LibAlchemica.getAvailableAlchemica(_realmId, i);
    }
  }
</pre>

- This function takes realmId as an argument and returns an array uint representing the available amount of Alchemicas in the Parcel.
<hr>

14. function calculateTransferAmounts
**Context**: When realm parcels are got, often times, there are _spillovers_. This refers to alchimcas that where did not go to parcel owner.
Each Realm parcel has its spill over rate.
<pre>
function calculateTransferAmounts(uint256 _amount, uint256 _spilloverRate) internal pure returns (TransferAmounts memory) {
    uint256 owner = (_amount * (bp - (_spilloverRate * 10**16))) / bp;
    uint256 spill = (_amount * (_spilloverRate * 10**16)) / bp;
    return TransferAmounts(owner, spill);
  }
</pre>

- The internal function takes a uint \_amount, and uint \_spilloverRate as argument to calculate the amount of Alchemica a user receives and the spillover amount.
- bp is a constant variable of 100 ether.
<hr>

15. function lastClaimedAlchemica
<pre>
function lastClaimedAlchemica(uint256 _realmId) external view returns (uint256) {
    return s.lastClaimedAlchemica[_realmId];
  }
</pre>

- This function takes realmId as an argument and returns the amount of last claimed alchemica.
<hr>

16. function claimAvailableAlchemica
**NB**: Users also get alchemica with their Aavegotchi NFTs.
<pre>
  function claimAvailableAlchemica(
    uint256 _realmId,
    uint256 _gotchiId,
    bytes memory _signature
  ) external gameActive {
    //Check signature
    require(
      LibSignature.isValid(keccak256(abi.encode(_realmId, _gotchiId, s.lastClaimedAlchemica[_realmId])), _signature, s.backendPubKey),
      "AlchemicaFacet: Invalid signature"
    );

    //1 - Empty Reservoir Access Right
    LibRealm.verifyAccessRight(_realmId, _gotchiId, 1, LibMeta.msgSender());
    LibAlchemica.claimAvailableAlchemica(_realmId, _gotchiId);
}

</pre>

**NB**: Some logics in the function were cut off for the sake of readability. This is with respect to this review file.

- This Function allows parcel owner to claim available alchemica with his parent NFT(Aavegotchi)
- It acheives this by taking realmId, gotchiId(NFT) and the signature message used for backend validation.
- Backend validation is acheived by calling isValid function from the LibSignature dependency. This function returns a bool value to validate or invalidate user.
- The function logic also confirms user right on the alchemica which is acheived by calling the verifyAccessRight on the LibRealm dependency.
- It also uses the claimAvailableAlchemica in LibAlchemica to calculate the amount of alchemica available for claim.
- **Alchemica claiming has an 8 hour cooldown time** before next claiming.
<hr>

17. function getHarvestRates
<pre>
function getHarvestRates(uint256 \_realmId) external view returns (uint256[] memory harvestRates) {
harvestRates = new uint256[](4);
for (uint256 i; i < 4; i++) {
harvestRates[i] = s.parcels[_realmId].alchemicaHarvestRate[i];
}
}
</pre>

- This function serves to return the harvest rate of a specified realm. It takes RealmId as an argument.
<hr>

18. function getCapacities
<pre>
function getCapacities(uint256 _realmId) external view returns (uint256[] memory capacities) {
    capacities = new uint256[](4);
    for (uint256 i; i < 4; i++) {
      capacities[i] = LibAlchemica.calculateTotalCapacity(_realmId, i);
    }
  }
</pre>

- Function takes realmId as an argument and returns the yield Alchemica capacity of the realm parcel.

19. function getTotalClaimed
<pre>
function getTotalClaimed(uint256 _realmId) external view returns (uint256[] memory totalClaimed) {
    totalClaimed = new uint256[](4);
    for (uint256 i; i < 4; i++) {
      totalClaimed[i] = LibAlchemica.getTotalClaimed(_realmId, i);
    }
  }
</pre>

_Refer to function getRoundAlchemica._

- This function gets total claimed Alchemica by taking the realmId as an argument and then subtracts the return value of getRoundAlchemica from alchemicaRemaining.
- It returns a uint array for each Alchemica's total claimed.

20. function channelAlchemica
**NB**<br>
Gotchus Alchemica(Alchemica) can be extracted through various ways. One which is channeling. This can come as a result of owning a gotchi or when installations are built on a realm parcel. They come as stipend for owning the gotchi or the installation.
<pre>
  function channelAlchemica(
    uint256 _realmId,
    uint256 _gotchiId,
    uint256 _lastChanneled,
    bytes memory _signature
  ) external gameActive {}
</pre>

- This implements the Alchemical channeling which allows users earn Alchemica for owning Aavegotchis.
- This function enables the transfer of Alchemica to the parent ERC721 token with id \_gotchiId and also to the great portal(i.e the portal that protects the Gotchiverse universe/Realm.)
- It takes input of realmId, gotchiId, \_lastChanneled and signature as arguments.
- Verification functions are called within this function before transfer is validated this includes Message signature used for backend validation.
- After transaction lastchenneled is updated to just concluded transaction.

21. getParcelLastChanneled
<pre>
/// @notice Return the last timestamp of an altar channeling
  /// @dev used as a parameter in channelAlchemica
  /// @param _parcelId Identifier of ERC721 parcel
  /// @return last channeling timestamp
  function getParcelLastChanneled(uint256 _parcelId) public view returns (uint256) {
    return s.parcelChannelings[_parcelId];
  }
</pre>
