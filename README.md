# PWAssist: PWA assistant

Small PHP class for automated PWA files creation : service-worker (javascript), manifest.json,
and generating icon set for all needed resoultions from a single image.

## Using

To start using PWAssist, just prepare configuration XML file with all needed parameters (optional), 
and create simple "maker" php file like this:

```php
// file : pwamake.php
include_once('../src/PWAssist.php');
PWAssist::init();
```
After that you can work in two ways:

- open this file in your browser, (the current folder must be somewhere inside *DocumentRoot* of your local web server software).
In that case you will see HTML page with three buttons for starting three tasks, and a form for editing and saving your PWA parameters (this will make/update pwa_config.xml file).
- or run pwamake.php from the shell:
> php pwamake.php {command} [{command2 ...}]

where {command} is one of manifest, icons, sw, all.
- sw - creates service worker js file
- manifest (or man) - creates manifest.json file
- icons - creates icon files for all your registered sizes. These files will be listed in manifest.json as icons for different resolutions.
- all - performs all these tasks in the following order: icons, manifest, sw.
So calling "all" is equivalent of "icons manifest sw".

## Configuration XML file

XML configuration file must have a name "pwa_config.xml" and be placed in the "current" folder with your pwamake.php
You can edit this file manuallty in text editor or enter all needed values in HTML form and press "Save parameters" button.
Here s a supported parameters list.
- __App id__ (in XML - appId) - Uniquie ID for your application. Must comply "variables naming" rules. Used to form cache name in case you leave it empty.
- __App Name__ (appName) - Application name ( value for __name__  in manifest.json)
- __App Short Name__ (appShortName) - short application name (value for __short_name__ in manifest.json)
- __App Description__ (appDesc) - application description (value for __description__ in manifest.json)
- __Start URI__ (appDesc) - application description (value used as __start_url__ in manifest.json and as a first item in precache file list inside SW script)
- __Home Folder__ : by default work performed on current folder, you started your pwamake.php from.
If you need to have another "document root" for your files, change default "./" to a right path (relative to current folder).
- __App Version__ (appVersion) - application current version (will be used in cacheName in service worker,
so after changing version newly generated SW script could force clearing from old cache and populate new one)
- __Lang__ (lang) will be used in manifest "lang" parameter.
- __SW template file__ (swTemplate) : here you can set your service-worker template file name, to override "standard" name "PWAssist.sw-template.js".
Note that if "standard template" does not exist, PWAssist uses js code builtin inside script.
But if you set your own name and such file won't be found in the folder with PWAssist.php,
service-worker file creating task will fail.
Standard template for service worker code is inside PWAssist.php module.
It is based on service worker code from Google developer examples (cache-First approach) and slightly modified
to be able insert "parameterized" blocks, like generated filelist to cache, cache name, string for detecting
"dynamic" URL that should be requested directly without caching.
You have to prepare "mustashed" macros inside your template to be replaced with final values.
For example, "{version}" will be replaced with App Version, "{cachename}" with cacheName parameter and so on.
{dynamicTest} will change to js code for checking for all "dynamicPart" items, so any matching will activate request to Server.
Finally, generated precached filelist will be inserted in place of "{filelist}" macro.

- __Service Worker Filename__ (swFilename) : name for the generated SW script file. Default is "service-worker.js"
- __Cached File Extensions__ (fileTypes) : comma separated extensions of files that will be pre-cached (included in file list in SW script).
- __Folders To Ignore__ (ignoreFolders) : folder names that won't be scanned for files to pre-cache.
- __Cached Files Size Limit__ (fileSizeLimit) : cached files size limit; all files bigger then this won't be cached.
Size can be set with "M" or "K" postfix : 2M is 2 MegaBytes (that's equivalent to 2097152 or 2048K), 100K is 100 KiloBytes.
- __Files To Ignore__ (ignoreFiles) : comma separated file mask for files to be excluded from cache file list.
- __Cache Name__ (cacheName) : string will be used with appVersion for dataCacheName and cacheName variables in SW script.
- __Source Icon File__ (baseIcon) : path to the source image that will be used to create icons in all listed resolutions.
If image with this name not found, "icons" task will be rejected.
- __All Icon Sizes__ (iconResolutions) : comma separated resoluitons for application icon.
- __Icon Filename Template__ (baseIcon) : string template for icon files, here "{size}" is a placeholder for image resolution (pix).
For example, if you set "img/myapp{size}.png", program wiill create folder "img" (if it does not exist yet), and image files
img/myapp48.png, img/myapp96.png etc.
- __Background Color__ (backgroundColor) and __Theme Color__ (themeColor) are just a values for "background_color" and "theme_color" in generated manifest.json
- __Dynamic Requests Contain__ (dynamicPart) : comma separated list of substrings that will say our service worker "I am dynamic request, don't cache me".
So when your application call GET request that includes one of these strings, SW will make direct network request.

## Source Image file for icons.
I hope you understand that "source" image for all icons must have resolution greater or equal to the most large icon.
Otherwise upscaling will create icons looking not so good.

PNG format is preferenced for source image, as it supports transparency.
Other source image types supported (gif, webp, jpg), but if you choose some ot them, PWAssist will create png icons from it,
probably without transparency areas.

## Demo
Working sample demonstrating "service page" with all buttons for creating PWA modules :
[demo/pwamake.php](demo/pwamake.php)

## License
Distributed under MIT License :
http://opensource.org/licenses/MIT
