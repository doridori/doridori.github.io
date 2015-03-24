---
layout: post
title: TableTest
---

|-----------------+------------+-----------------+----------------|
| Default aligned |Left aligned| Center aligned  | Right aligned  |
|-----------------|:-----------|:---------------:|---------------:|
| First body part |Second cell | Third cell      | fourth cell    |
| Second line     |foo         | **strong**      | baz            |
| Third line      |quux        | baz             | bar            |
|-----------------+------------+-----------------+----------------|
| Second body     |            |                 |                |
| 2 line          |            |                 |                |
|=================+============+=================+================|
| Footer row      |            |                 |                |
|-----------------+------------+-----------------+----------------|

The approach I am using is

	(docDpi / 72) = px per point size = ppps
	ppps / androidDensity = PtMultiplier
	PtMultiplier * docPtSize = AndroidPx
	AndroidPx = AndroidDp


| PS doc dpi* | PS doc res | designing FOR Android density | PS Pt size | Formula            | Android Dp/Sp  |
|-------------|------------|-------------------------------|------------|--------------------|----------------|
| 72dpi       | 320x480    | MD (160 dpi) 1x               | 10         | (72/72) * 1 * 10   | 10             |
| 72dpi       | 480x720    | HD (240 dpi) 1.5x             | 15         | (72/72) / 1.5 * 15 | 10             |
| 72dpi       | 480x960    | XHD (320 dpi) 2x              | 20         | (72/72) / 2 * 20   | 10             |
| 160dpi      | 320x480    | MD (160dpi) 1x                | 10         | (160/72) / 1 * 10  | 22.2           |
| 320dpi      | 640x960    | XHD (320dpi) 2x               | 10         | (320/72) / 2 * 10  | 22.2           |

