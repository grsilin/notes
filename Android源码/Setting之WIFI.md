# Setting/wifi

## conect to wifi

***

## tetherwifi

turn on a hotspot,share network with other devices

*source code:TetherWifiSetting.java,layout:ether_wifi_prefs*

use broadcast monitor `hotspotclientchanged` ,`wpscheckpinfailed`,`hotspotoverlap`,`apstatechanged` action.

when receive the state is changed:

1. wifi ap client changed:

    update category

2. wifi ap state changed:
    