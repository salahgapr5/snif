# DISPENSATION MODULE

## Overview
- The module allows a user to crete an Airdrop. Which accepts a input and output list 
- This transaction needs to signed by atleast all addresses of the input list ( can be set to more ,but not less)
- The module accumulates  funds from  the input address list and distributes it among the output list .
- The whole process happens in one block .


## Technicals 
### Data structures
 - The base-level data structure is 
```go
type Distribution struct {
	DistributionType DistributionType `json:"distribution_type"`
	DistributionName string           `json:"distribution_name"`
}
```
This is stored in the keeper with the key DistributionType_DistributionName for historical records. Therefore, the combination of type and name needs to be unique

- Distribution records are created for processing individual transfers to recipients
```go
type DistributionRecord struct {
	DistributionName string         `json:"distribution_name"`
	RecipientAddress sdk.AccAddress `json:"recipient_address"`
	Coins            sdk.Coins      `json:"coins"`
}
```
This record is also stored in the keeper for historical records .

### User flow 
 The set of user commands to use this module 
```shell
#Multisig Key - It is a key composed of two or more keys (N) , with a signing threshold (K) ,such that the transaction needs K out of N votes to go through.

#create airdrop
#mkey        : multisig key
#ar1         : name of airdrop , needs to be unique for every airdrop. If not the tx gets rejected
#input.json  : list of funding addresses  -  Input address must be part of the multisig key
#output.json : list of airdrop receivers.

sifnodecli tx dispensation create mkey ar1 input.json output.json --generate-only >> offlinetx.json

#First user signs
sifnodecli tx sign --multisig $(sifnodecli keys show mkey -a) --from $(sifnodecli keys show sif -a)  offlinetx.json >> sig1.json

#Second user signs
sifnodecli tx sign --multisig $(sifnodecli keys show mkey -a) --from $(sifnodecli keys show akasha -a)  offlinetx.json >> sig2.json

#Multisign created from the above signatures
sifnodecli tx multisign offlinetx.json mkey sig1.json sig2.json >> signedtx.json

#transaction broadcast , distribution happens
sifnodecli tx broadcast signedtx.json
```

### Events Emitted 
Transfer events are emitted for each transfer . There are two type of transfers in a distribution
- Transfer for address in the input list to the Dispensation Module Address.

```json
"type": "transfer",
"attributes": [
{
"key": "recipient",
"value": "sif1zvwfuvy3nh949rn68haw78rg8jxjevgm2c820c"
},
{
"key": "sender",
"value": "sif1syavy2npfyt9tcncdtsdzf7kny9lh777yqc2nd"
},
{
"key": "amount",
"value": "15000000000000000000rowan"
}
```
- Transfer of funds from the Dispensation Module Address to recipients in the output list
```json
"type": "transfer",
"attributes": [
{
"key": "recipient",
"value": "sif1p6z0ze9mztfd8cx5z9g6pndmzdrtxnsfesnn97"
},
{
"key": "sender",
"value": "sif1zvwfuvy3nh949rn68haw78rg8jxjevgm2c820c"
},
{
"key": "amount",
"value": "10000000000000000000rowan"
}
```


- A distribution complete event is emitted at the end of a distribution .
```json
 {
  "type": "distribution_complete",
  "attributes": [
    {
      "key": "module_account",
      "value": "sif1zvwfuvy3nh949rn68haw78rg8jxjevgm2c820c"
    }
  ]
}
```


### Queries supported
```shell
#Query all distributions
sifnodecli q dispensation distributions-all
#Query distribution records by name
sifnodecli q dispensation records-by-name ar1
#Query distribution records by address
sifnodecli q dispensation records-by-addr sif1cp23ye3h49nl5ty35vewrtvsgwnuczt03jwg00
```