# TechOpt Home Assistant blueprints

Blueprints from the Home Assistant setups written about on
[techopt.io/tag/home-assistant](https://www.techopt.io/tag/home-assistant).

Files live under `blueprints/automation/techoptio/`, which is the path Home
Assistant uses for imported automations. Use the badge beside a blueprint, or
copy the YAML into that folder on your instance.

## Lightning notification

Two automations, used together. They expect a strike feed that publishes
distance in kilometers, direction in degrees, and a counter that increases by
one for each strike.
[Blitzortung](https://github.com/mrk-its/homeassistant-blitzortung) does this.
These blueprints do not install that integration, and they do not notify on
every strike the counter sees.

Keep the close radius equal to the distance band's inner edge. With the
defaults, 1 km through 15 km is one alert, and anything under 1 km is the
capped close alerts. A strike at exactly 1 km stays with the band.

### Distance band

[![Import Lightning distance band](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Ftechoptio%2Fhomeassistant%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Ftechoptio%2Flightning-distance-band.yaml)

One alert when a strike first enters the band, then silence for a cooldown
(default 2 hours). A later strike that is still inside the band does not
match, because the previous distance is not beyond the outer edge. The
automation mode is `single`, so a trigger during the cooldown is discarded
rather than queued. A restart ends the cooldown early.

### Close strikes

[![Import Lightning close strikes](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Ftechoptio%2Fhomeassistant%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Ftechoptio%2Flightning-close-strikes.yaml)

Each new strike inside the close radius can notify, up to a cap (default 3)
in a window that starts with the first of those alerts (default 60 minutes).
The window does not slide. The fourth and later close strikes in that window
are silent. The next close strike after the window expires opens a new one,
sets the count back to 1, and notifies.

Create these two helpers first:

- An **input number** for the count. Minimum 0, step 1, maximum at least the
  alert cap.
- An **input datetime** with both date and time, for when the window started.

The close blueprint triggers on the counter. It reads the distance sensor
when the condition runs, not from the trigger. That matches a feed which
updates azimuth, then the counter, then distance in one handler: Home
Assistant schedules the counter trigger and returns, so the distance update
lands before the condition. A strike outside the close radius can still
increment the counter. It does not use one of the alerts.

### The sentence

Both blueprints build the same sentence. Distance is converted to meters and
rounded to the nearest meter.

- 1000 meters or more: one decimal kilometer. Exactly 1 is `1 kilometer`. A
  whole number of kilometers drops the decimal. Anything else keeps one
  decimal, such as `1.4 kilometers`.
- 0 meters after rounding: `less than a meter`.
- 1 meter: `1 meter`.
- Anything else under 1000 meters: the meter count, such as `900 meters`.

Direction uses eight names: north, northeast, east, southeast, south,
southwest, west, northwest. The index is `((degrees + 22.5) / 45)` as an
integer, modulo 8. The place is that name, then `of`, then the place name
(default `home`). If the direction is not a number, the place is `from`
plus that name.

`Lightning strike 900 meters southwest of home.`

The notification action is yours. `title` and `message` are set just before
it runs. Default title: `Lightning nearby`.

## License

[MIT](LICENSE)
