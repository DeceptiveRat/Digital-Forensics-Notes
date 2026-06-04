## 1. browser history
- artifacts:
	- website visits
	- cache
	- searches
	- downloads
	- timestamps
	- Chromium apps
	- file location and formats

## 2. website visits
- data retention per browser:
	- Chromium based browsers store up to 90 days of website history
	- Mozilla Firefox stores maximum number of website visits, depending on computer performance
	- Apple Safari stores up to 1 year
- information recorded:
	- date/time of visit
	- title of webpage
	- URL of webpage
- visit types:
	- link
	- typed
	- bookmark
	- other: e.g. redirect
- visit source:
	- synced: synchronized from different device
	- extension: added by extension
	- imported: imported from different browser
- local files opened via Windows Explorer are also logged 

## 3. cache
- cached images:
	- metadata such as URL is also stored
	- common for images to be hosted on different domain from website
	- images may be loaded by advertisements in webpage
- html file may be cached 

## 4. downloads
- recorded information:
	- download URL 
	- local save path
	- date/time
	- number of bytes downloaded
- Chrome and MS Edge also store:
	- URL of web page
	- whether user opened file from browser

## 5. Chrome history
- location:
	- Windows:
		- `C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default`
		- `C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default\Cache`
	- macOS:
		- `/Users/<username>/Library/Application Support/Google/Chrome/Default`
		- `/Users/<username>/Library/Caches/Google/Chrome/Default/Cache`
	- Linux:
		- `/home/<username>/.config/google-chrome/Default`
		- `/home/<username>/.cache/google-chrome/Default/Cache`

- format:
	- bookmarks: *Bookmarks* JSON file
	- browser settings: *Preferences* JSON file
	- cache: *index* file, *data_#* file, and *f_######* files
	- cookies: *Cookies* SQLite database
	- downloads: *History* SQLite database
	- favicons: *Favicons* SQLite database
	- form history: *Web Data* SQLite database
	- logins: *Login Data* SQLite database
	- searches: *History* SQLite database
	- session data: *Session* and *Tabs* files
	- thumbnails: *Top Sites* SQLite database
	- website visits: *History* SQLite database

## 6. MS Edge
- location:
	- Windows:
		- `C:\Users\<username>\AppData\Local\Microsoft\Edge\User Data\Default`
		- `C:\Users\<username>\AppData\Local\Microsoft\Edge\User Data\Default\Cache`
	- macOS:
		- `/Users/<username>/Library/Application Support/Microsoft Edge/Default`
		- `/Users/<username>/Library/Caches/Microsoft Edge/Default/Cache`

- format:
	- bookmarks: *Bookmarks* JSON file
	- browser settings: *Preferences* JSON file
	- cache: *index* file, *data_#* file, and *f_######* files
	- cookies: *Cookies* SQLite database
	- downloads: *History* SQLite database
	- favicons: *Favicons* SQLite database
	- logins: *Login Data* SQLite database
	- searches: *History* SQLite database
	- session data: *Current Session*, *Current Tabs*, *Last Session*, and *Last Tabs* files
	- thumbnails: *Top Sites* SQLite database
	- website visits: *History* SQLite database

## 7. Mozilla Firefox
- location:
	- Windows:
		- `C:\Users\<username>\AppData\Roaming\Mozilla\Firefox\Profiles\<profile folder>`
		- `C:\Users\<username>\AppData\Local\Mozilla\Firefox\Profiles\<profile folder>\cache2`
	- macOS:
		- `/Users/<username>/Library/Application Support/Firefox/Profiles/<profile folder>`
		- `/Users/<username>/Library/Caches/Firefox/Profiles/<profile folder>/cache2`
	- Linux:
		- `/home/<username>/.mozilla/firefox/<profile folder>`
		- `/home/<username>/.cache/mozilla/firefox/<profile folder>/cache2`
- format:
	- bookmarks: *places.sqlite*
	- browser settings: *prefs.js*
	- cache: *_CACHE_MAP_*, *_CACHE_00#_*, and data files
	- cookies: *cookies.sqlite*
	- downloads: *places.sqlite*
	- favicons: *favicons.sqlite*
	- form history: *formhistory.sqlite*
	- logins: *logins.json*
	- session data: *sessionstore.jsonlz4*
	- thumbnails: *thumbnails* directory
	- website visits: *places.sqlite*

## reference
[1] Introduction to Web Browser Forensics, 2026/04/02, https://www.foxtonforensics.com/web-browser-forensics
[2] Browser History Examiner - User Guide, 2026/04/02, https://www.foxtonforensics.com/browser-history-examiner/chrome-history-location
[3] Browser History Examiner - User Guide, 2026/04/02, https://www.foxtonforensics.com/browser-history-examiner/microsoft-edge-history-location
[4] Browser History Examiner - User Guide, 2026/04/02, https://www.foxtonforensics.com/browser-history-examiner/firefox-history-location 
