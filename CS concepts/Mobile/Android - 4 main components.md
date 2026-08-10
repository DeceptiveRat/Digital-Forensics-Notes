## activities
- manages user interface
- must inherit `Activity` class and redefine lifecycle method

## services
- runs in background regardless of certain activities
- used for services that have a long life time
- must inherit `Service` class and redefine lifecycle method
- e.g. music playing after screen turns off

## broadcast receivers
- reacts to certain broadcasts
- no user interface
- broadcasts:
	- timezone change
	- insufficient battery
	- language change
- sleeps after reacting to certain broadcast
- only guarantees 10 seconds of runtime; use service or separate thread for longer features
- must inherit `BroadcastReceiver` and redefine `onReceive()` method
- e.g. insufficient battery pop-up when battery low

## content providers
- provides standardized interface between applications
- actual data provided may be in file system or data base

## reference
[1] (Charlie_moon, 2021/07/15) 안드로이드 4대 컴포넌트 Main Components of Android, 2026/08/03, https://charlie-dev.tistory.com/3