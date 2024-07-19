+++
title = "Identifiants d'appareil"
description = "En apprendre plus sur identifiants matériels"
weight = 2
+++

When connecting with the same key_material on different devices, thoses devices exchanges their hardaware fingerprint to check wether they are allowed to connect.
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
- **pending**: this device is waiting for an authorisiation: user have to review this device to allow or reject it.

