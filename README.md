# GTEX bulk data download

## Get object ID for each sample

Sample ID has the following format: `GTEX-12BJ1-1226-SM-5LUAE`. The list of relevant sample
IDs is in a file called `sample_list.txt`

```bash
$ cat sample_list.txt
GTEX-12BJ1-1226-SM-5LUAE
GTEX-13N11-0426-SM-5KM3O
GTEX-13N11-0326-SM-5LUA3
GTEX-144GM-1526-SM-79OJM
GTEX-145ME-0526-SM-5QGQV
GTEX-145ME-2026-SM-5SIA5
GTEX-15CHR-0726-SM-7EPHG
GTEX-15CHR-1226-SM-79OON
GTEX-15EOM-0126-SM-7KUGG
GTEX-15RIE-1726-SM-7KUMU
GTEX-15RIE-2426-SM-7KUDS
GTEX-15SB6-0726-SM-6M48F
GTEX-16BQI-1526-SM-6LLIE
GTEX-1CAMR-1926-SM-7EPI2
GTEX-1CAMS-1426-SM-7IGPM
GTEX-1HSMQ-2126-SM-CJI3E
GTEX-1HSMQ-0726-SM-B2LXY
GTEX-1HSMQ-1326-SM-ADEHS
GTEX-1HSMQ-1826-SM-A9SMJ
GTEX-1HSMQ-1226-SM-CGQFX
GTEX-1I1GU-1126-SM-A96S9
GTEX-1ICG6-2026-SM-CE6SX
GTEX-1ICG6-0626-SM-ACKWS
GTEX-1MCC2-2126-SM-EWRON
GTEX-1R9PN-0326-SM-DTX85
```

`file-manifest.json` was downloaded from GTEX. It contains all the files available for 
download. The file name is prefixed by the sample ID, and the file includes bigWig files,
BAM files, and BAM indexes (BAI files).

I wrote a script to parse the file manifest that retrieves the record for each file:
file name, md5sum, file size, and object ID.
Then I filtered the records to include only the samples in `sample_list.txt` and selected
only BAM files (in this example there was only 1 BAM per sample in this case).

```bash
python idquery.py < sample_list.txt > sample_table.txt
```

The data is saved in the Google Spreadsheet an in the file `sample_table.txt`



## Download the gen3 data client

```bash
wget https://github.com/uc-cdis/cdis-data-client/releases/download/1.0.0/dataclient_linux.zip
unzip dataclient_linux.zip
```

## Configure gen3

First need to log in to gen3 data commons using eRA commons ID.
[https://gen3.theanvil.io/identity](https://gen3.theanvil.io/identity)

Once logged in you have the opportunity to generate an API key. 
The API key is saved as `credentials.json`.

Configure the gen3 client like this:

```bash
gen3-client configure --profile=bendall --cred=credentials.json --apiendpoint=https://gen3.theanvil.io

## Download one sample

After configuration, the command to download one sample looks like this:

```
./gen3-client download-single --no-prompt --skip-completed --profile=bendall --protocol=s3 --guid=$(basename $GUID)
```

(We call "basename $GUID" because the object IDs are )

## Download all samples and check MD5

```bash
cat sample_table.txt | while read SAMP FNAME MD5 FSIZE GUID; do
    ./gen3-client download-single --no-prompt --skip-completed --profile=bendall --protocol=s3 --guid=$(basename $GUID)
    echo "$MD5 $FNAME" | md5sum -c - 
done
```
