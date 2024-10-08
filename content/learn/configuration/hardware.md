+++
title = "Hardware Key"
description = "Learn more about the hardware keys"
weight = 1
+++

When connecting with the same key_material on different devices, those devices exchanges their hardware fingerprint to check wether they are allowed to connect.
This add an extra layer of security in the unlucky case where your secret material is shared by another person on the internet (which could be relatively frequent as users tends use weak passwords).

Those hardware keys are managed by the system entity: 
```js
sys{
    AllowedHardware{
        name: String,
        status: String, 
    }
}
```

**name** provides the name of the device.

**status** can have three values:
- **enabled** : your device can connect to this other device, 
- **disabled**: your device cannot connect to this device,
- **pending**: this device is waiting for an authorization: user have to review this device to allow or reject it.
