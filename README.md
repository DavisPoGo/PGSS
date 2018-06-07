# PGSS ( Pokemon Go Screenshot Scanner) 
PGSS scans raid near by images and identifies Gym,Raid Egg/Boss and time and then updates monocle hydro database. PGSS also works as backend for RealDeviceRaidMap. Most of gym images are identified automatically.

## Features
1. Read raid near by sighting images and identify
	* gym 
	* raid boss
	* start time
2. Parameters to identify gym and raid boss are stored in `gym_images` and `pokemon_images` table automatically.
3. Update raids and fort_sightings tables in monocle (Hydro) database
4. Download gym(fort) url images and find matching gym automatically. Up to 99% of gyms are detected successfully.
5. Discord bot to download user submitted screenshot in your discord server 
6. MySQL and Postgresql supported.

## Requirements
* Python 3.6
* Tesseract
* Linux/macOS. Never tested on Windows.
* macOS and iOS required for RealDeviceRaidMap(<https://github.com/123FLO321/RealDeviceRaidMap/>)

## How it works
### raidnearby.py
Read all raid near by screen shot image cropped by `crop.py` in `process_img` directory and extract gym/raid boss/hatch time information and update `raids` and `fort_sightings` table for monocle Hydro database. If raidnearby.py can't identify the gym then the gym image is stored in `unknown_img` as `FortImage_xxx.png`. Once gym is identified, check level and time. If time is Ongoing(Raid), then try to identify raid boss by checking with `pokemon_images` table. If the raid boss is unknown, then store the raid boss image into `non_find_img` as PokemonImage_xxx.png.

### findfort.py
Read all gym images in `unknown_img` and identify the gym(fort) image by comparing fort URL images in `url_img`. If findfort.py finds matching gym(fort) in `url_img`, then update `gym_images` table to set identified `fort_id`. findfort.py checks images every 30 seconds. Fort URL images need to be downloaded by `downloadfortimg.py` before running `findfort.py`.

### downloadfortimg.py
Download all fort URL images in `Forts` table. Set `MAP_START` and `MAP_END` in `config.py` to limit fort URL images to download if you want.

### manualsubmit.py
`manualsubmit.py` update `fort_id` in `gym_images` and `pokemon_id` in `pokemon_images` by reading `Fort_xxx.png` and `Pokemon_yyy.png` in `not_find_img`. User need to set xxx for `fort_id` and yyy for `pokedex id` manually. This part need to be integrated with `Frontend` of RealDeviceRaidMap in the future.

### Running order
1. Copy config.example.py and rename to config.py. Configure config.py based on your setup.
2. Run `python3.6 downloadfortimg.py` once to download all gym(fort) URL images
3. Run `python3.6 raidscan.py` start raid iamge scanning
4. Run `python3.6 manualsubmit.py` once you set `Fort_xxx.png` and `Pokemon_yyy.png` in `not_find_img` directory.

## Setting up
1. Install Python 3.6
 * macOS : I download from here <https://www.python.org/downloads/release/python-365/> and install
 * Linux (Ubuntu example)
    ```
    apt-get install build-essential
    sudo add-apt-repository ppa:jonathonf/python-3.6
    apt-get update
    sudo apt-get install python3.6 python3.6-dev
    wget https://bootstrap.pypa.io/get-pip.py
    sudo python3.6 get-pip.py
    ```
    There are many other way to install python3.6. Google it.
3. Install tesseract 
 * macOS : `brew install tesseract`
 * Linux
    ```
    apt-get update
    sudo apt-get install tesseract-ocr
    ```
4. Create venv
    `python3.6 -m venv path/to/create/venv`
	example: `python3.6 -m venv ~/venv_pgss`
5. Activate venv
    `source ~/venv_pgss/bin/activate`
6. Install requirements
    `pip3.6 install -r requirements.txt -U`
    * If you don't have MySQL on your machine, comment out mysqlclient
    * If you don't have Postgresql on your machine, commment out psycopg2 and psycopg2-binary
7. Configure config.py to set your monocle database and set discord server setting if you use rssbot.py 
8. Run `python3.6 downloadfortimg.py`. If you don't want to download whole fort images in database, set `MAP_START` and `MAP_END` in `config.py`.
9. Run `python3.6 raidscan.py` from the command line. When first run, raid_images and pokemon_images tables are added automatically.
10. **Note. If you were running crop.bash for Frontend of ReadDevicePokeMap, stop crop.bash before running raidscan.py. raidscan.py itself gets screenshot image and crop with crop.py**. Don't worry, PGSS can identify gym images up to 99% of gyms automatically (without user input).
11. Run `python3.6 rssbot.py` to start downloading user posted screenshot image on your discord server
12. Wait until all gyms are identified. Check `success_img` and `need_check_img` directory to make sure all gym images are correctly identified.
13. `PokemonImage_xxx.png` files are stored in `unknown_img` directory. Rename the file to `Pokemon_PokemonId.png`(e.g. `Pokemon_380.png` for Latias) and run `python3.6 manualsubmit.py`. This will train pokemon raid boss. Usually only one time training should be enough.
14. If screenshot image size is not in config.py save the iamge to `not_find_img` as `Image_aaaxbbb.png` and you have to configure `RAID_NEARBY_SIZE`.

