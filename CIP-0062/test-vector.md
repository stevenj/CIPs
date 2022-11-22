# Test Vector for CIP-0062

## Delegation

### Keys
`payment verification key`:
```
{
    "type": "PaymentVerificationKeyShelley_ed25519",
    "description": "Payment Verification Key",
    "cborHex": "58203bc3383b1b88a628e6fa55dbca446972d5b0cd71bcd8c133b2fa9cd3afbd1d48"
}
``` 

`payment secret key`: 
```
{
    "type": "PaymentSigningKeyShelley_ed25519",
    "description": "Payment Signing Key",
    "cborHex": "5820b5c85fa8fb2d8cd4e4f624c206946652b6764e1af83034a79b32320ce3940dd9"
}
```

`staking verification key`:
```
{
    "type": "StakeVerificationKeyShelley_ed25519",
    "description": "Stake Verification Key",
    "cborHex": "5820b5462be6a8a8ec0c4d6ee6edb83794a03df1bca43edc72b380df2ad3a982a555"
}
```

`staking secret key`:
```
{
    "type": "StakeSigningKeyShelley_ed25519",
    "description": "Stake Signing Key",
    "cborHex": "58202f669f45365099666940922d47b29563d2c9f885c88a077bfea17631a7579d65"
}
```

### Addresses


### Delegation Certificate

`Delegation certificate sample`:
```
{
  "1":[["1788b78997774daae45ae42ce01cf59aec6ae2acee7f7cf5f76abfdd505ebed3",1],["b48b946052e07a95d5a85443c821bd68a4eed40931b66bd30f9456af8c092dfa",3]],
  "2":"93bf1450ec2a3b18eebc7acfd311e695e12232efdf9ce4ac21e8b536dfacc70f",
  
  "3":"e1160a9d8f375f8e72b4bdbfa4867ca341a5aa6f17fde654c1a7d3254e",
  "4":5479467,
  "5":0
}
```

`Delegation certificate after signature`:
```
{
  "61284":{
    "1":[["1788b78997774daae45ae42ce01cf59aec6ae2acee7f7cf5f76abfdd505ebed3",1],["b48b946052e07a95d5a85443c821bd68a4eed40931b66bd30f9456af8c092dfa",3]],
    "2":"93bf1450ec2a3b18eebc7acfd311e695e12232efdf9ce4ac21e8b536dfacc70f",
    "3":"e1160a9d8f375f8e72b4bdbfa4867ca341a5aa6f17fde654c1a7d3254e",
    "4":5479467,
    "5":0
  },
  "61285":"0x3c25da29d43e70fb331c93b1197863e0d0a2e1cf7048994c580b0fc974f16bbb18c389aee380a66c0e7b6141f1df77b5db132dc228dbae9167238d96d4c4a80a"
}
```

## Voting 

TODO: Once Voting is fully spec-ed we can add this

### Proposal and Choices


### Vote Fragment / Transaction
