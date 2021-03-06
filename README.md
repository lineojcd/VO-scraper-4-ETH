# vo-scraper

A python script for ETH students to download lecture videos from [video.ethz.ch](https://video.ethz.ch/).

Copied from https://gitlab.ethz.ch/tgeorg/vo-scraper

## Requirements:
 * `requests`

Install with:

    pip3 install requests

## Setup
Download the file [here](https://gitlab.ethz.ch/tgeorg/vo-scraper/raw/master/vo-scraper.py?inline=false) and run with

    python3 vo-scraper.py

# FAQ

### Q: How do I use it?

#### A:

    python3 vo-scraper.py <arguments> <lecture link(s)>

To see a list of possible arguments check

    python3 vo-scraper.py --help

**For protected lectures** the vo-scraper will ask for your login credentials before downloading the video(s).

### Q: How do I pass a file with links to multiple lectures?

#### A: Use `--file <filename>`

The file should have a single link for each new line. Lines starting with `#` will be ignored and can be used for comments. It should look something like this:

    https://video.ethz.ch/lectures/<department>/<year>/<spring/autumn>/XXX-XXXX-XXL.html
    # This is a comment
    https://video.ethz.ch/lectures/<department>/<year>/<spring/autumn>/XXX-XXXX-XXL.html
    ...

Additionally you can also add a username and password at the end of the link seperated by a single space:

    https://video.ethz.ch/lectures/<department>/<year>/<spring/autumn>/XXX-XXXX-XXL.html username passw0rd1
    ...

**Note:** This is **NOT** recommended for your NETHZ account password for security reasons!

### <a name="how_it_works"></a> Q: How does it acquire the videos?

#### A: Like so:

Each lecture on [video.ethz.ch](https://video.ethz.ch/) has a JSON file with metadata associated with it.

So for example

    https://video.ethz.ch/lectures/d-infk/2019/spring/252-0028-00L.html

has its JSON file under:

    https://video.ethz.ch/lectures/d-infk/2019/spring/252-0028-00L.series-metadata.json

This JSON file contains a list of all "episodes" where the ids of all the videos of the lecture are located.

Using those ids we can access another JSON file with the video's metadata under

    https://video.ethz.ch/.episode-video.json?recordId=<ID>

Example:

    https://video.ethz.ch/.episode-video.json?recordId=3f6dee77-396c-4e2e-a312-a41a457b319f

This file contains links to all available video streams (usually 1080p, 720p, and 360p). Note that if a lecture requires a login, this file will only be accessible if you a cookie with a valid login-token!

The link to the video stream looks something like this:

    https://oc-vp-dist-downloads.ethz.ch/mh_default_org/oaipmh-mmp/<video id>/<video src id?>/presentation_XXXXXXXX_XXXX_XXXX_XXXX_XXXXXXXXXXXX.mp4

Example:

    https://oc-vp-dist-downloads.ethz.ch/mh_default_org/oaipmh-mmp/3f6dee77-396c-4e2e-a312-a41a457b319f/2bd93636-e95d-4552-8722-332a95e1a0a6/presentation_c6539ed0_1af9_490d_aec0_a67688dad755.mp4

So what the vo-scraper does is getting the list of episodes from the lecture's metadata and then acquiring the links to the videos selected by the user by accessing the videos' JSON files. Afterwards it downloads the videos behind the links.

### Q: How does it access lecture videos that are password protected?

#### A: Like so:

There exist three (known) types of protection for lecture videos:

* `NONE` requires no login at all

* `ETH` requires logging in with a NETHZ account

* `PWD` requires logging in with a custom user name and password

What kind of protection a series of lecture videos has, can be found in its metadata file under `<lecture link>.series-metadata.json`, e.g.

    https://video.ethz.ch/lectures/d-infk/2019/spring/252-0028-00L.series-metadata.json

This JSON file has a field called `protection` with a value corresponding to one of the three protection types.

If the series is protected then a cookie containing an authentication token needs to be sent when requesting the individual videos' metadata file at `https://video.ethz.ch/.episode-video.json?recordId=<ID>`

Getting a cookie with a valid token differs between videos that require a NETHZ login and videos that use custom credentials.

For NETHZ logins we need to send a POST request to `https://video.ethz.ch/j_security_check` with the following headers:

    Content-Type: application/x-www-form-urlencoded
    CSRF-Token: undefined
    User-Agent: Mozilla/5.0

as well as the following parametres:

    __charset__: utf-8
    j_validate: True
    j_username: <NETHZ username>
    j_password: <NETHZ password>

For logins with custom credentials we have to perforn a POST request to `<lecture link>.series-login.json`, e.g.:

    https://video.ethz.ch/lectures/d-infk/2020/spring/252-0220-00L.series-login.json

with the following headers:

    Referer: <lecture link>.html
    User-Agent: Mozilla/5.0

as well as the following parametres:

    __charset__: utf-8
    username: <custom username>
    password: <custom password>

In both cases we get back a cookie which we then can include when requesting the individual video metdata files.

### Q: It doesn't work for my lecture. What can I do to fix it?

#### A: Follow these steps:
1. Make sure you have connection to [video.ethz.ch](https://video.ethz.ch/). The scraper should let you know when there's no connection.
2. Try running it again. Sometimes random issues can throw it off.
3. If the lecture is password protected, make sure you use the correct credentials. Most protected lectures require your NETHZ credentials while some use a custom username and password.
4. Make sure you're running the newest version of the scraper by re-downloading the script from the repository. There might have been an update.
5. Check whether other lectures still work. Maybe the site was updated which broke the scraper.
6. Enable the debug flag with `-v` and see whether any of the additional information now provided is helpful.
7. Check "[How does it acquire the videos?](#how_it_works)" and see whether you can manually reach the video in your browser following the steps described there.
8. After having tried all that without success, feel free to open up a new issue. Make sure to explain what you have tried and what the results were. There is no guarantee I will respond within reasonable time as I'm a busy student myself. If you can fix the issue yourself, feel free to open a merge request with the fix.


### Q: Can you fix *X*? Can you implement feature *Y*?

#### A: Feel free to open an issue [here](https://gitlab.ethz.ch/tgeorg/vo-scraper/issues). Merge requests are always welcome but subject to my own moderation.
***

Loosely based on https://gitlab.ethz.ch/dominik/infk-vorlesungsscraper
