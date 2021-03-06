tiger-delta
===========

Makes a map of what was remapped in TIGER between 2006 and the present.


Making the 2015 vector base map
-------------------------------

    mkdir /mnt/tiger
    cd /mnt/tiger
    git clone https://github.com/mapbox/tippecanoe.git
    git clone https://github.com/ericfischer/tiger-delta.git
    wget -m -np http://www2.census.gov/geo/tiger/TIGER2015/ROADS/
    wget -m -np http://www2.census.gov/geo/tiger/TIGER2015/FEATNAMES/
    find www2.census.gov/geo/tiger/TIGER2015/ROADS/ -name '*.zip' -print | xargs -L1 -P8 unzip
    mkdir data
    mv tl* data
    cd data
    find . -name '*.shp' -print | xargs -t -L1 -P8 sh -c 'ogr2ogr -f GeoJSON $(basename $1 .shp).json $1' sh
    cd ../tiger-delta
    make dbfcat
    find ../www2.census.gov/geo/tiger/TIGER2015/FEATNAMES -name '*.zip' -print | xargs -t -L1 -P8 sh -c 'unzip -p $1 $(basename $1 .zip).dbf | ./dbfcat | ./handle-abbreviations > ../data/$1.abbrev' sh
    find ../data -name '*.json' -print | sort | xargs -L1 -t ./expand-abbreviations | ../tippecanoe/tippecanoe -f -Z12 -z12 -d13 -pr -l tiger2015 -o ../tiger2015.mbtiles

Requires about 30GB of disk space plus tippecanoe temporary storage. I used c3.2xlarge.

Making the 2016 vector base map
-------------------------------

```
mkdir /mnt/data/tiger
cd /mnt/data/tiger
git clone https://github.com/mapbox/tippecanoe.git
git clone https://github.com/ericfischer/tiger-delta.git
wget -e robots=off -m -np http://www2.census.gov/geo/tiger/TIGER2016/ROADS/
wget -e robots=off -m -np http://www2.census.gov/geo/tiger/TIGER2016/FEATNAMES/
mkdir data
cd data
find ../www2.census.gov/geo/tiger/TIGER2016/ROADS/ -name '*.zip' -print | xargs -L1 -P0 unzip
find . -name '*.shp' -print | xargs -t -L1 -P0 sh -c 'ogr2ogr -f GeoJSON $(basename $1 .shp).json $1' sh
cd ../tiger-delta/
make dbfcat
find ../www2.census.gov/geo/tiger/TIGER2016/FEATNAMES -name '*.zip' -print | xargs -t -L1 -P0 sh -c 'unzip -p $1 $(basename $1 .zip).dbf | ./dbfcat | ./handle-abbreviations > ../data/$1.abbrev' sh
cd ../tippecanoe/
sudo apt-get install libsqlite3-dev
make -j
cd ../tiger-delta/
find ../data -name '*.json' -print | sort | xargs -L1 -t ./expand-abbreviations | ../tippecanoe/tippecanoe -f -Z12 -z12 -d13 -l tiger2016 -o ../tiger2016.mbtiles
```

Required 28GB of disk space plus Tippecanoe temporary storage. I used c3.2xlarge.
