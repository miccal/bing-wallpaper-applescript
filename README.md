# Set The Bing Image Of The Day As Your Desktop Wallpaper

This AppleScript fetches the Bing image of the day using any valid market code and sets it as your desktop wallpaper.

This AppleScript can then be added to the `Shortcuts.app` in macOS Monterey and run from the Menu Bar.

## Background Information

Using the Japanese market code `ja-JP` as an example, the direct download link for the Bing image of the day can be found using the `url`

`http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=ja-JP`

where:

* `format` can take the values `js` (short for `json`), `xml` and `rss`;
* `idx` is the number days previous the present day, with `0` meaning the current day;
* `n` is the number of images to fetch previous to the day given by `idx`, with `1` meaning fetch the one image for day `idx`; and
* `mkt` is the market code. There are currently a total of 38 market codes as listed [here](https://docs.microsoft.com/en-us/bing/search-apis/bing-web-search/reference/market-codes). At present, only the market codes `de-DE`, `en-AU`, `en-CA`, `en-GB`, `en-IN`, `en-US`, `es-ES`, `fr-CA`, `fr-FR`, `ja-JP` and `zh-CN` have their own localised versions. Other market codes are set as being the “Rest of the World” (with the generic market code `ROW`).

## Details

The AppleScript basically works by first downloading the Bing image of the day and saving it to `~/Downloads/bing_image_of_the_day.jpg`, then setting this image as the desktop wallpaper.

Again using the Japanese market code `ja-JP` as an example, the Bing image of the day is downloaded with the following one-liner CLI command
```
curl --silent "http://www.bing.com/$(curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=ja-JP" | grep --only-matching "\"url\":\"\/.*.jpg&pid=hp" | sed 's/"url":"\///g')" > ~/Downloads/bing_image_of_the_day.jpg
```

&nbsp;

Let's unpack what this one-liner CLI command does ...

### Step 1

The command

```
curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=ja-JP"
```

outputs the information for the current image of the day, like so:

`{"images":[{"startdate":"20211110","fullstartdate":"202111101500","enddate":"20211111","url":"/th?id=OHR.BeaversBend_JA-JP2539821984_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp","urlbase":"/th?id=OHR.BeaversBend_JA-JP2539821984","copyright":"ビーバーズ・ベンド・リゾートパーク, 米国 オクラホマ州 （© Inge Johnsson/Alamy Stock Photo）","copyrightlink":"https://www.bing.com/search?q=%E3%83%93%E3%83%BC%E3%83%90%E3%83%BC%E3%82%BA%E3%83%BB%E3%83%99%E3%83%B3%E3%83%89%E3%83%BB%E3%83%AA%E3%82%BE%E3%83%BC%E3%83%88%E3%83%91%E3%83%BC%E3%82%AF&form=hpcapt&filters=HpDate%3a%2220211110_1500%22","title":"ブロークン・ボウ湖の紅葉","quiz":"/search?q=Bing+homepage+quiz&filters=WQOskey:%22HPQuiz_20211110_BeaversBend%22&FORM=HPQUIZ","wp":true,"hsh":"be331c8a5be95ef8d4585717b1e70865","drk":1,"top":1,"bot":1,"hs":[]}],"tooltips":{"loading":"読み込み中...","previous":"前の画像へ","next":"次の画像へ","walle":"この画像を壁紙としてダウンロードすることはできません。","walls":"この画像をダウンロードできます。画像の用途は壁紙に限定されています。"}}`

### Step 2

The next part, namely

`grep --only-matching "\"url\":\"\/.*.jpg&pid=hp"`

then pulls out the `url` location from the previous output, so that the command

```
curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=ja-JP" | grep --only-matching "\"url\":\"\/.*.jpg&pid=hp"
```

outputs

`"url":"/th?id=OHR.BeaversBend_JA-JP2539821984_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp`

### Step 3

The next part, namely

`sed 's/"url":"\///g'`

then removes the `"url":"/` part of the previous output, so that the command

```
curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=ja-JP" | grep --only-matching "\"url\":\"\/.*.jpg&pid=hp" | sed 's/"url":"\///g'
```

outputs

`th?id=OHR.BeaversBend_JA-JP2539821984_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp`

### Step 4

We then have the required direct download link, namely

`http://www.bing.com/th?id=OHR.BeaversBend_JA-JP2539821984_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp`

and the last step is to download the image and save it as `~/Downloads/bing_image_of_the_day.jpg`, which is accomplished with the final one-liner CLI command shown above.

To use this one-liner CLI command in AppleScript, it is necessary to escape some characters (namely the `"`'s and `\`'s). The following is an AppleScript that sends this one-liner CLI command to `Terminal.app`:
```
tell application "Terminal"
	do script "curl --silent \"http://www.bing.com/$(curl --silent \"http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=ja-JP\" | grep --only-matching \"\\\"url\\\":\\\"\\/.*.jpg&pid=hp\" | sed 's/\"url\":\"\\///g')\" > ~/Downloads/bing_image_of_the_day.jpg" in front window
end tell
```

## The Full AppleScript

```
set market_code_choices to {"da-DK, Denmark, Danish", "de-AT, Austria, German", "de-CH, Switzerland, German", "de-DE, Germany, German", "en-AU, Australia, English", "en-CA, Canada, English", "en-GB, United Kingdom, English", "en-ID, Indonesia, English", "en-IN, India, English", "en-MY, Malaysia, English", "en-NZ, New Zealand, English", "en-PH, Republic of the Philippines, English", "en-US, United States, English", "en-ZA, South Africa, English", "es-AR, Argentina, Spanish", "es-CL, Chile, Spanish", "es-ES, Spain, Spanish", "es-MX, Mexico, Spanish", "es-US, United States, Spanish", "fi-FI, Finland, Finnish", "fr-BE, Belgium, French", "fr-CA, Canada, French", "fr-CH, Switzerland, French", "fr-FR, France, French", "it-IT, Italy, Italian", "ja-JP, Japan, Japanese", "ko-KR, Korea, Korean", "nl-BE, Belgium, Dutch", "nl-NL, Netherlands, Dutch", "no-NO, Norway, Norwegian", "pl-PL, Poland, Polish", "pt-BR, Brazil, Portuguese", "ru-RU, Russia, Russian", "sv-SE, Sweden, Swedish", "tr-TR, Turkey, Turkish", "zh-CN, Peoples Republic of China, Chinese", "zh-HK, Hong Kong SAR, Traditional Chinese", "zh-TW, Taiwan, Traditional Chinese"}

set market_code to choose from list market_code_choices with prompt "Market code (Code, Country/Region, Language):" default items {"ja-JP, Japan, Japanese"}

if market_code is false then return

delay 3

tell application "System Events"
	tell every desktop
		set picture to "~/Downloads/no_image.jpg"
	end tell
end tell

delay 3

tell application "Terminal"
	do script "curl --silent \"http://www.bing.com/$(curl --silent \"http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=$(echo " & market_code & " | grep --extended-regexp --only-matching \"[a-z]{2}-[A-Z]{2}\")\" | grep --only-matching \"\\\"url\\\":\\\"\\/.*.jpg&pid=hp\" | sed 's/\"url\":\"\\///g')\" > ~/Downloads/bing_image_of_the_day.jpg" in front window
end tell

delay 3

tell application "System Events"
	tell every desktop
		set picture to "~/Downloads/bing_image_of_the_day.jpg"
	end tell
end tell
```

&nbsp;

Let's unpack what this AppleScript does ...

### Step 1

The first part of the AppleScript, namely

```
set market_code_choices to {"da-DK, Denmark, Danish", "de-AT, Austria, German", "de-CH, Switzerland, German", "de-DE, Germany, German", "en-AU, Australia, English", "en-CA, Canada, English", "en-GB, United Kingdom, English", "en-ID, Indonesia, English", "en-IN, India, English", "en-MY, Malaysia, English", "en-NZ, New Zealand, English", "en-PH, Republic of the Philippines, English", "en-US, United States, English", "en-ZA, South Africa, English", "es-AR, Argentina, Spanish", "es-CL, Chile, Spanish", "es-ES, Spain, Spanish", "es-MX, Mexico, Spanish", "es-US, United States, Spanish", "fi-FI, Finland, Finnish", "fr-BE, Belgium, French", "fr-CA, Canada, French", "fr-CH, Switzerland, French", "fr-FR, France, French", "it-IT, Italy, Italian", "ja-JP, Japan, Japanese", "ko-KR, Korea, Korean", "nl-BE, Belgium, Dutch", "nl-NL, Netherlands, Dutch", "no-NO, Norway, Norwegian", "pl-PL, Poland, Polish", "pt-BR, Brazil, Portuguese", "ru-RU, Russia, Russian", "sv-SE, Sweden, Swedish", "tr-TR, Turkey, Turkish", "zh-CN, Peoples Republic of China, Chinese", "zh-HK, Hong Kong SAR, Traditional Chinese", "zh-TW, Taiwan, Traditional Chinese"}

set market_code to choose from list market_code_choices with prompt "Market code (Code, Country/Region, Language):" default items {"ja-JP, Japan, Japanese"}

if market_code is false then return
```

opens up a list of market codes for selection:

<p align="center">
<img width="500" alt="Screen Shot 2021-11-12 at 23 11 59" src="https://user-images.githubusercontent.com/6127163/141489460-d1d774b5-e71e-41e3-bdf2-0b758796e911.png">
</p>

and selecting `Cancel` exits the AppleScript.

For the above example, the variable `market_code` is then set to `ja-JP, Japan, Japanese`.

### Step 2

The next part of the AppleScript

```
tell application "System Events"
	tell every desktop
		set picture to "~/Downloads/no_image.jpg"
	end tell
end tell
```

just removes the current wallpaper image.

### Step 3

The next part of the AppleScript

```
tell application "Terminal"
	do script "curl --silent \"http://www.bing.com/$(curl --silent \"http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=$(echo " & market_code & " | grep --extended-regexp --only-matching \"[a-z]{2}-[A-Z]{2}\")\" | grep --only-matching \"\\\"url\\\":\\\"\\/.*.jpg&pid=hp\" | sed 's/\"url\":\"\\///g')\" > ~/Downloads/bing_image_of_the_day.jpg" in front window
end tell
```

then sends the one-liner CLI command to `Terminal.app`, where the (suitably escaped for AppleScript) command

```
echo " & market_code & " | grep --extended-regexp --only-matching \"[a-z]{2}-[A-Z]{2}\"
```

just pulls out the market code from the `market_code` variable.

For example, the CLI command

```
echo ja-JP, Japan, Japanese | grep --extended-regexp --only-matching "[a-z]{2}-[A-Z]{2}"
```

simply outputs

`ja-JP`

### Step 4

The final part of the AppleScript

```
tell application "System Events"
	tell every desktop
		set picture to "~/Downloads/bing_image_of_the_day.jpg"
	end tell
end tell
```

then sets the downloaded image `~/Downloads/bing_image_of_the_day.jpg` as the desktop wallpaper.

## Adding the AppleScript to the `Shortcuts.app`

In the `Shortcuts.app`, create a new Shortcut

<p align="center">
<img width="200" alt="Screen Shot 2021-11-12 at 23 32 08" src="https://user-images.githubusercontent.com/6127163/141492985-18725826-7081-448d-ae2d-3d45fe127e37.png">
</p>

that performs a `Run AppleScript` action

<p align="center">
<img width="800" alt="Screen Shot 2021-11-12 at 23 32 47" src="https://user-images.githubusercontent.com/6127163/141493216-f8233ca0-5825-42f9-a70a-284839c6759e.png">
</p>

that may then be run from the Manu Bar

<p align="center">
<img width="500" alt="Screen Shot 2021-11-12 at 23 32 55" src="https://user-images.githubusercontent.com/6127163/141493274-393e6b87-4d0b-4d25-b4f6-27a254cc24ad.png">
</p>

## License

The Unlicense (Public Domain, essentially).
