# Set The Bing Image Of The Day As Your Desktop Wallpaper

This AppleScript fetches the Bing image of the day using any choice of a valid market code and sets it as your desktop wallpaper.

The AppleScript can then be added to the `Shortcuts.app` in macOS Monterey and run from the Menu Bar.

Using the Japanese market code `ja-JP` as an example, the direct download link for the Bing image of the day can be found from

`http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=ja-JP`

where:

* `format` can take the values `js` (short for `json`), `hp` (short for `html`), `xml` (the default) and `rss`;
* `uhd` can take the values `0` and `1`, with `0` meaning fetch the standard resolution version of the image (normally a resolution of 1920x1080) and `1` meaning fetch the ultra high definition resolution version of the image (which can vary from a minimum resolution of 1920x1080 -- for example, the image obtained in the example below is 5349×3009); 
* `idx` is the number days previous to the present day, with `0` meaning the present day;
* `n` is the number of images to fetch previous to the day given by `idx`, with `1` meaning fetch the one image for day `idx`; and
* `mkt` is the market code. There are currently a total of 38 market codes as listed [here](https://docs.microsoft.com/en-us/bing/search-apis/bing-web-search/reference/market-codes), and at present (August 2023) only the market codes `de-DE`, `en-CA`, `en-GB`, `en-IN`, `en-US`, `es-ES`, `fr-CA`, `fr-FR`, `it-IT`, `ja-JP`, `ko-KR` (sometimes), `pt-BR` and `zh-CN` have their own localised versions. Other market codes are set as being the “Rest of the World” (with the generic market code `ROW`).

## Details

The AppleScript basically works by first downloading the Bing image of the day and saving it to `~/Downloads/bing_image_of_the_day.jpg`, then setting this image as the desktop wallpaper.

Again using the Japanese market code `ja-JP` as an example, the Bing image of the day can be downloaded with the following one-liner CLI command
```
curl --silent "http://www.bing.com/$(curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=ja-JP" | grep --only-matching "\"url\":\"\/.*\.jpg" | sed 's/"url":"\///g')" > ~/Downloads/bing_image_of_the_day.jpg
```

&nbsp;

Let's unpack what this one-liner CLI command does ...

### Step 1

The command

```
curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=ja-JP"
```

outputs the information for the current image of the day, like so:

`{"images":[{"startdate":"20211120","fullstartdate":"202111201500","enddate":"20211121","url":"/th?id=OHR.ElephantGiving_JA-JP6387498046_UHD.jpg&rf=LaDigue_UHD.jpg&pid=hp&w=1920&h=1080&rs=1&c=4","urlbase":"/th?id=OHR.ElephantGiving_JA-JP6387498046","copyright":"アフリカゾウの家族, ケニア （© Yva Momatiuk and John Eastcott/Minden Pictures）","copyrightlink":"https://www.bing.com/search?q=%E3%82%A2%E3%83%95%E3%83%AA%E3%82%AB%E3%82%BE%E3%82%A6+%E5%AE%B6%E6%97%8F&form=hpcapt&filters=HpDate%3a%2220211120_1500%22","title":"今日は「家族の日」","quiz":"/search?q=Bing+homepage+quiz&filters=WQOskey:%22HPQuiz_20211120_ElephantGiving%22&FORM=HPQUIZ","wp":true,"hsh":"1870702a47d30be6d7aee03f41b36604","drk":1,"top":1,"bot":1,"hs":[]}],"tooltips":{"loading":"読み込み中...","previous":"前の画像へ","next":"次の画像へ","walle":"この画像を壁紙としてダウンロードすることはできません。","walls":"この画像をダウンロードできます。画像の用途は壁紙に限定されています。"}}`

### Step 2

The next part, namely

`grep --only-matching "\"url\":\"\/.*\.jpg"`

then pulls out relevant part of the `url` location from the previous output, so that the command

```
curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=ja-JP" | grep --only-matching "\"url\":\"\/.*\.jpg"
```

outputs

`"url":"/th?id=OHR.ElephantGiving_JA-JP6387498046_UHD.jpg&rf=LaDigue_UHD.jpg`

### Step 3

The next part, namely

`sed 's/"url":"\///g'`

then removes the `"url":"/` part of the previous output, so that the command

```
curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=ja-JP" | grep --only-matching "\"url\":\"\/.*\.jpg" | sed 's/"url":"\///g'
```

outputs

`th?id=OHR.ElephantGiving_JA-JP6387498046_UHD.jpg&rf=LaDigue_UHD.jpg`

### Step 4

We then have the required direct download link, namely

`http://www.bing.com/th?id=OHR.ElephantGiving_JA-JP6387498046_UHD.jpg&rf=LaDigue_UHD.jpg`

and the last step is to download the image and save it as `~/Downloads/bing_image_of_the_day.jpg`, which is accomplished with the final one-liner CLI command shown above, and the resultant image is illustrated below:

<p align="center">
<img width="2000" alt="th" src="https://user-images.githubusercontent.com/6127163/142742880-4e556123-1983-4c70-aa05-64e16654063e.jpeg">
</p>

Note that the `&rf=LaDigue_UHD.jpg` part of the direct download link is a "fallback" image if the actual image `ElephantGiving_JA-JP6387498046_UHD.jpg` is not available. So if the downloaded image is the one illustrated below, something is wrong!

<p align="center">
<img width="2000" alt="th" src="https://user-images.githubusercontent.com/6127163/142700897-71e19683-b2e5-4e06-8083-56f332d0716a.png">
</p>

To use this one-liner CLI command in AppleScript, it is necessary to escape some characters (namely the `"`'s and `\`'s). The following is an AppleScript that sends this one-liner CLI command to `Terminal.app`:
```
tell application "Terminal"
	do script "curl --silent \"http://www.bing.com/$(curl --silent \"http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=ja-JP\" | grep --only-matching \"\\\"url\\\":\\\"\\/.*\\.jpg\" | sed 's/\"url\":\"\\///g')\" > ~/Downloads/bing_image_of_the_day.jpg" in front window
end tell
```

## The Full AppleScript

<p align="center">
<img width="2000" alt="Screen Shot 2021-11-20 at 07 17 02" src="https://user-images.githubusercontent.com/6127163/142703032-e562914c-f3d7-427b-abe4-e8b9d0c99fd0.png">
</p>

&nbsp;

Let's unpack what this AppleScript does ...

### Step 1

The first part of the AppleScript, namely

```
set market_code_choices to {"da-DK, Denmark, Danish", "de-AT, Austria, German", "de-CH, Switzerland, German", "de-DE, Germany, German", "en-AU, Australia, English", "en-CA, Canada, English", "en-GB, United Kingdom, English", "en-ID, Indonesia, English", "en-IN, India, English", "en-MY, Malaysia, English", "en-NZ, New Zealand, English", "en-PH, Republic of the Philippines, English", "en-US, United States, English", "en-ZA, South Africa, English", "es-AR, Argentina, Spanish", "es-CL, Chile, Spanish", "es-ES, Spain, Spanish", "es-MX, Mexico, Spanish", "es-US, United States, Spanish", "fi-FI, Finland, Finnish", "fr-BE, Belgium, French", "fr-CA, Canada, French", "fr-CH, Switzerland, French", "fr-FR, France, French", "it-IT, Italy, Italian", "ja-JP, Japan, Japanese", "ko-KR, Korea, Korean", "nl-BE, Belgium, Dutch", "nl-NL, Netherlands, Dutch", "no-NO, Norway, Norwegian", "pl-PL, Poland, Polish", "pt-BR, Brazil, Portuguese", "ru-RU, Russia, Russian", "sv-SE, Sweden, Swedish", "tr-TR, Turkey, Turkish", "zh-CN, Peoples Republic of China, Chinese", "zh-HK, Hong Kong SAR, Traditional Chinese", "zh-TW, Taiwan, Traditional Chinese"}

set market_code to choose from list market_code_choices with prompt "Market code (Code, Country/Region, Language):" default items {"ja-JP, Japan, Japanese"}

if market_code is false then return
```

opens up a list of valid market codes for selection:

<p align="center">
<img width="500" alt="Screen Shot 2021-11-12 at 23 11 59" src="https://user-images.githubusercontent.com/6127163/141489460-d1d774b5-e71e-41e3-bdf2-0b758796e911.png">
</p>

and selecting `Cancel` exits the AppleScript.

For the above example, the market code `ja-JP` is selected and the variable `market_code` is then set to `ja-JP, Japan, Japanese`.

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
	do script "curl --silent \"http://www.bing.com/$(curl --silent \"http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=$(echo " & market_code & " | grep --extended-regexp --only-matching \"[a-z]{2}-[A-Z]{2}\")\" | grep --only-matching \"\\\"url\\\":\\\"\\/.*\\.jpg\" | sed 's/\"url\":\"\\///g')\" > ~/Downloads/bing_image_of_the_day.jpg" in front window
end tell
```

then sends the one-liner CLI command to `Terminal.app`, where the (suitably escaped for AppleScript) command

`echo " & market_code & " | grep --extended-regexp --only-matching \"[a-z]{2}-[A-Z]{2}\"`

just pulls out the market code from the `market_code` variable.

For example, the CLI command

```
echo ja-JP, Japan, Japanese | grep --extended-regexp --only-matching "[a-z]{2}-[A-Z]{2}"
```

simply outputs

`ja-JP`

### Step 4

The next part of the AppleScript

```
tell application "System Events"
	tell every desktop
		set picture to "~/Downloads/bing_image_of_the_day.jpg"
	end tell
end tell
```

then sets the downloaded image `~/Downloads/bing_image_of_the_day.jpg` as the desktop wallpaper.

### Step 5

The final part of the AppleScript

```
tell application "Terminal"
	do script "curl --silent \"http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=$(echo " & market_code & " | grep --extended-regexp --only-matching \"[a-z]{2}-[A-Z]{2}\")\" | grep --only-matching \"\\\"url\\\":\\\"\\/.*\\.jpg\" | sed 's/\"url\":\"\\/th?id=OHR\\.//g' | sed 's/\\.jpg.*//g'" in front window
end tell
```

provides a simple check of the file name, market code and image resolution obtained by the one-liner CLI command.

For example, the CLI command

```
curl --silent "http://www.bing.com/HPImageArchive.aspx?format=js&uhd=1&idx=0&n=1&mkt=$(echo ja-JP, Japan, Japanese | grep --extended-regexp --only-matching "[a-z]{2}-[A-Z]{2}")" | grep --only-matching "\"url\":\"\/.*\.jpg" | sed 's/"url":"\/th?id=OHR\.//g' | sed 's/\.jpg.*//g'
```

simply outputs

`ElephantGiving_JA-JP6387498046_UHD`

## Adding the AppleScript to the `Shortcuts.app`

In the `Shortcuts.app`, create a new Shortcut

<p align="center">
<img width="200" alt="Screen Shot 2021-11-12 at 23 32 08" src="https://user-images.githubusercontent.com/6127163/141492985-18725826-7081-448d-ae2d-3d45fe127e37.png">
</p>

that performs a `Run AppleScript` action with the above AppleScript code

<p align="center">
<img width="800" alt="Screen Shot 2021-11-12 at 23 32 47" src="https://user-images.githubusercontent.com/6127163/141493216-f8233ca0-5825-42f9-a70a-284839c6759e.png">
</p>

that may then be run from the Menu Bar

<p align="center">
<img width="500" alt="Screen Shot 2021-11-12 at 23 32 55" src="https://user-images.githubusercontent.com/6127163/141493274-393e6b87-4d0b-4d25-b4f6-27a254cc24ad.png">
</p>

## License

The Unlicense (Public Domain, essentially).
