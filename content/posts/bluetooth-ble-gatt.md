---
title: "Bluetooth BLE GATT"
date: 2025-03-30T09:56:16+08:00
draft: false
---

如何给一个Iot设备配置网络呢？使用蓝牙看起来是一个可行的方案。低功耗蓝牙可以以极少的电量，
长期处于广播状态，广播自己的服务（service）和特征（characteristic）。

蓝牙协议定义了很多标准的服务和特征，比如设备信息等。每个服务和特征都有自己的uuid，自定义时，
应避免和标准的冲突。

## Golang BLE GATT server

golang有一些ble的库，但大部分都在最近不再维护了。`tinygo.org/x/bluetooth` 仍然在维护，支持
Linux/Windows/嵌入式平台，以下是这个库中的几个example，展示了获取设备信息，读取电池电量。

```go
// example that demonstrates how to create a BLE peripheral device with the Device Information Service.
package main

import (
	"time"

	"tinygo.org/x/bluetooth"
)

var adapter = bluetooth.DefaultAdapter

var (
	localName = "TinyGo Device"

	manufacturerName               = "TinyGo"
	manufacturerNameCharacteristic bluetooth.Characteristic

	modelNumber               = "Model 1"
	modelNumberCharacteristic bluetooth.Characteristic

	serialNumber               = "123456"
	serialNumberCharacteristic bluetooth.Characteristic

	softwareVersion               = "1.0.0"
	softwareVersionCharacteristic bluetooth.Characteristic

	firmwareVersion               = "1.0.0"
	firmwareVersionCharacteristic bluetooth.Characteristic

	hardwareVersion               = "1.0.0"
	hardwareVersionCharacteristic bluetooth.Characteristic
)

func main() {
	println("starting")
	must("enable BLE stack", adapter.Enable())
	adv := adapter.DefaultAdvertisement()
	must("config adv", adv.Configure(bluetooth.AdvertisementOptions{
		LocalName:    localName,
		ServiceUUIDs: []bluetooth.UUID{bluetooth.ServiceUUIDDeviceInformation},
	}))
	must("start adv", adv.Start())

	must("add service", adapter.AddService(&bluetooth.Service{
		UUID: bluetooth.ServiceUUIDDeviceInformation,
		Characteristics: []bluetooth.CharacteristicConfig{
			{
				Handle: &manufacturerNameCharacteristic,
				UUID:   bluetooth.CharacteristicUUIDManufacturerNameString,
				Value:  []byte(manufacturerName),
				Flags:  bluetooth.CharacteristicReadPermission,
			},
			{
				Handle: &modelNumberCharacteristic,
				UUID:   bluetooth.CharacteristicUUIDModelNumberString,
				Value:  []byte(modelNumber),
				Flags:  bluetooth.CharacteristicReadPermission,
			},
			{
				Handle: &serialNumberCharacteristic,
				UUID:   bluetooth.CharacteristicUUIDSerialNumberString,
				Value:  []byte(serialNumber),
				Flags:  bluetooth.CharacteristicReadPermission,
			},
			{
				Handle: &softwareVersionCharacteristic,
				UUID:   bluetooth.CharacteristicUUIDSoftwareRevisionString,
				Value:  []byte(softwareVersion),
				Flags:  bluetooth.CharacteristicReadPermission,
			},
			{
				Handle: &firmwareVersionCharacteristic,
				UUID:   bluetooth.CharacteristicUUIDFirmwareRevisionString,
				Value:  []byte(firmwareVersion),
				Flags:  bluetooth.CharacteristicReadPermission,
			},
			{
				Handle: &hardwareVersionCharacteristic,
				UUID:   bluetooth.CharacteristicUUIDHardwareRevisionString,
				Value:  []byte(hardwareVersion),
				Flags:  bluetooth.CharacteristicReadPermission,
			},
		},
	}))

	for {
		time.Sleep(time.Second)
	}
}

func must(action string, err error) {
	if err != nil {
		panic("failed to " + action + ": " + err.Error())
	}
}
```

```go
// example that demonstrates how to create a BLE peripheral device with the Battery Service.
package main

import (
	"math/rand"
	"time"

	"tinygo.org/x/bluetooth"
)

var adapter = bluetooth.DefaultAdapter

var (
	localName = "TinyGo Battery"

	batteryLevel uint8 = 75 // 75%
	battery      bluetooth.Characteristic
)

func main() {
	println("starting")
	must("enable BLE stack", adapter.Enable())
	adv := adapter.DefaultAdvertisement()
	must("config adv", adv.Configure(bluetooth.AdvertisementOptions{
		LocalName:    localName,
		ServiceUUIDs: []bluetooth.UUID{bluetooth.ServiceUUIDBattery},
	}))
	must("start adv", adv.Start())

	must("add service", adapter.AddService(&bluetooth.Service{
		UUID: bluetooth.ServiceUUIDBattery,
		Characteristics: []bluetooth.CharacteristicConfig{
			{
				Handle: &battery,
				UUID:   bluetooth.CharacteristicUUIDBatteryLevel,
				Value:  []byte{byte(batteryLevel)},
				Flags:  bluetooth.CharacteristicReadPermission | bluetooth.CharacteristicNotifyPermission,
			},
		},
	}))

	for {
		println("tick", time.Now().Format("04:05.000"))

		// random variation in batteryLevel
		batteryLevel = randomInt(65, 85)

		// and push the next notification
		battery.Write([]byte{batteryLevel})

		time.Sleep(time.Second)
	}
}

func must(action string, err error) {
	if err != nil {
		panic("failed to " + action + ": " + err.Error())
	}
}

// Returns an int >= min, < max
func randomInt(min, max int) uint8 {
	return uint8(min + rand.Intn(max-min))
}
```

## 手机调试

下载 `nRF Connect` 可以方便扫描和查看服务和特征，也可以读写特征。
