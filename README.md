# openHAB-virtual-alarm-clock
An virtual alarm clock with openHAB.

## Items

You need:

* a switch item for activating the alarm clock
* a number item for setting the hours
* a number item for setting the minutes

```
Group gVirtualAlarmClock "Virtual Alarm Clock" <time>

Switch virtual_alarm_clock "Alarm Clock" <clock> (gVirtualAlarmClock)
Number virtual_alarm_clock_H "Alarm Clock Hour [%s]" <calendar> (gVirtualAlarmClock)
Number virtual_alarm_clock_M "Alarm Clock Minute [%s]" <calendar> (gVirtualAlarmClock)
```

## Rules

The rule is cron based and will trigger if the switch is enabled:

```
var Timer alarm_clock = null

rule "alarm clock"
    when
        Time cron "0 * * * * ?"
    then
        if (virtual_alarm_clock.state == ON) {
                var target_minute = (virtual_alarm_clock_M.state as DecimalType).intValue
                var target_hour = (virtual_alarm_clock_H.state as DecimalType).intValue

                var time_now = now()

                if (target_minute == time_now.getMinute() && target_hour == time_now.getHour()) {
                    <play_music_item>.sendCommand("<file>.mp3")
                    <volume_item>.sendCommand(30)
                    alarm_clock = createTimer(now.plusMinutes(3)) [|
                        <stop_music_item>.sendCommand(ON);
            ]
                }
        }
end



rule "alarm clock off"
when
        Item virtual_alarm_clock changed to OFF
then
        <stop_music_item>.sendCommand(ON);
end
```

You need:

* an `<play_music_item>`. For me this is a item for a Sonos speaker which can play a `URI` using the [Sonos Binding](https://www.openhab.org/addons/bindings/sonos/).
* a `<file>.mp3`. This is the `URI` my Sonos speaker will play.
* an `<volume_item>`. This is an item which will control the volume of my Sonos speaker.
* an `<stop_music_item>`. This is an item which will stop the Sonos speaker playing the `URI` file.

## Sitemaps

Inside your sitemap you have to add:

```
Text label="virtual alarm clock" icon="time" {
    Switch item=virtual_alarm_clock mappings=[ON="Alarm clock on", OFF="Alarm clock off"]
    Setpoint item=virtual_alarm_clock_H minValue=0 maxValue=23 step=1
    Setpoint item=virtual_alarm_clock_M minValue=0 maxValue=59 step=1
}
```

This will give you the possibility to

* activate the alarm clock.
* deactivate the alarm clock.
* manually deactivate the alarm.
* set the hours for the alarm clock.
* set the minutes for the alarm clock.
