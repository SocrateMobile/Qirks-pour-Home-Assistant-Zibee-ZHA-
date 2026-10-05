# Qirks-pour-Home-Assistant-Zibee-ZHA-

voici différents Qirks pour des capteur TUYA Zigbee

[![Buy Me A Coffee](https://img.buymeacoffee.com/button-api/?text=Buy+me+a+coffee&emoji=☕&slug=Socrate&button_colour=FFDD00&font_colour=000000&font_family=Poppins&outline_colour=000000&coffee_colour=ffffff)](https://www.buymeacoffee.com/Socrate)

# Tuya TS0601 mmWave presence sensor _TZE284_iadro9bf.

from __future__ import annotations

from enum import Enum
from typing import cast

from zigpy import types as t
from zigpy.quirks import CustomDevice
from zigpy.zcl.clusters.measurement import OccupancySensing
from zigpy.zcl.clusters.general import Basic, Groups, Ota, Scenes, Time, GreenPowerProxy

from zhaquirks.tuya import TuyaLocalCluster, TuyaNewManufCluster
from zhaquirks.tuya.builder import TuyaQuirkBuilder


def _dp_to_occupancy(value: int) -> int:
    """Convert Tuya DP to ZCL OccupancySensing.occupancy."""
    return 1 if int(value) == 1 else 0


class IhsenoPresenceOccupancy(OccupancySensing, TuyaLocalCluster):
    """Occupancy cluster backed by Tuya presence datapoints."""


class TuyaMMWaveSensor(CustomDevice):
    """Custom device representing Tuya mmWave sensor with endpoint 242."""

    signature = {
        "models_info": [("_TZE284_iadro9bf", "TS0601")],
        "endpoints": {
            # Endpoint 1 principal
            1: {
                "profile_id": 0x0104,
                "device_type": 0x0051,
                "input_clusters": [
                    Basic.cluster_id,
                    Groups.cluster_id,
                    Scenes.cluster_id,
                    0xed00,
                    TuyaNewManufCluster.cluster_id,
                ],
                "output_clusters": [
                    Time.cluster_id,
                    Ota.cluster_id,
                ],
            },
            # Endpoint 242 Green Power
            242: {
                "profile_id": 0xA1E0,
                "device_type": 0x0061,
                "input_clusters": [],
                "output_clusters": [
                    0x0021, # GreenPowerProxy.cluster_id
                ],
            },
        },
    }

    replacement = {
        "endpoints": {
            1: {
                "profile_id": 0x0104,
                "device_type": 0x0107, # OCCUPANCY_SENSOR
                "input_clusters": [
                    Basic.cluster_id,
                    Groups.cluster_id,
                    Scenes.cluster_id,
                    0xed00,
                    TuyaNewManufCluster,
                    IhsenoPresenceOccupancy, # Ajout de notre cluster
                ],
                "output_clusters": [
                    Time.cluster_id,
                    Ota.cluster_id,
                ],
            },
            242: {
                "profile_id": 0xA1E0,
                "device_type": 0x0061,
                "input_clusters": [],
                "output_clusters": [
                    0x0021,
                ],
            },
        },
    }


(
    TuyaQuirkBuilder(TuyaMMWaveSensor)
    .adds(IhsenoPresenceOccupancy)
    
    # DP 1 : Présence
    .tuya_dp(
        dp_id=1,
        ep_attribute=IhsenoPresenceOccupancy.ep_attribute,
        attribute_name=OccupancySensing.AttributeDefs.occupancy.name,
        converter=_dp_to_occupancy,
    )
    
    # DP 102 : Sensibilité
    .tuya_number(
        dp_id=102,
        attribute_name="sensitivity",
        fallback_name="Sensibilité",
        min_value=1,
        max_value=10,
        translation_key="sensitivity",
    )
    
    # DP 105 : Distance maximale
    .tuya_number(
        dp_id=105,
        attribute_name="max_range",
        fallback_name="Distance Maximale",
        min_value=0,
        max_value=1000,
        unit="cm",
        translation_key="max_range",
    )
    
    .add_to_registry()
)
''''


''''
"""Tuya TS0601 human presence sensor _TZE284_debczeci."""

from __future__ import annotations

from enum import Enum
from typing import cast

from zigpy import types as t
from zigpy.zcl.clusters.measurement import OccupancySensing

from zhaquirks.tuya import TuyaLocalCluster
from zhaquirks.tuya.builder import TuyaQuirkBuilder

def _true_false_0_to_occupancy(value: int) -> int:
    """Convert Tuya 'trueFalse0' to ZCL OccupancySensing.occupancy."""
    return 1 if int(value) == 0 else 0

class PirSensitivity(t.enum8):
    """PIR sensor sensitivity."""
    low = 0
    middle = 1
    high = 2

class PirTime(t.enum8):
    """PIR delay time in seconds."""
    s15 = 0
    s30 = 1
    s60 = 2

class IhsenoPresenceOccupancy(OccupancySensing, TuyaLocalCluster):
    """Occupancy cluster backed by Tuya presence datapoints."""

(
    TuyaQuirkBuilder("_TZE284_debczeci", "TS0601")
    # On force la reconnaissance de la signature exacte vue dans ton diagnostic
    .applies_to("_TZE284_debczeci", "TS0601")
    # On ajoute le cluster de présence
    .adds(IhsenoPresenceOccupancy)
    
    # DP 1: presence (0=mouvement, 1=pas de mouvement)
    .tuya_dp(
        dp_id=1,
        ep_attribute=IhsenoPresenceOccupancy.ep_attribute,
        attribute_name=OccupancySensing.AttributeDefs.occupancy.name,
        converter=_true_false_0_to_occupancy,
    )
    
    # DP 4: batterie (0-100%)
    .tuya_battery(
        dp_id=4,
        scale=2,
    )
    
    # DP 9: Sensibilité PIR
    .tuya_enum(
        dp_id=9,
        attribute_name="pir_sensitivity",
        enum_class=cast(type[Enum], PirSensitivity),
        translation_key="pir_sensitivity",
        fallback_name="PIR sensitivity",
    )
    
    # DP 10: Délai PIR
    .tuya_enum(
        dp_id=10,
        attribute_name="pir_time",
        enum_class=cast(type[Enum], PirTime),
        translation_key="pir_time",
        fallback_name="PIR delay",
    )
    
    # Cette option force l'ajout même si la signature ne correspond pas à 100% à un profil standard
    .add_to_registry(force_add_cluster=True)
)


''''
