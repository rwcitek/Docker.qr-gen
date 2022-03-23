# Generating QR codes within Docker

## Run an interactive container
```bash
# run an interactive session in the background
docker run --rm --name qrgen rwcitek/barcode-gen sleep inf & sleep 1

# exec into it
docker exec -it qrgen /bin/bash
```

## Example 1
```bash
# generate UUID and qrcode
uuidgen | tee uuid.txt | qrencode --type=PNG --level=H --output uuid.qrcode.png

# resize png to reasonable size
convert uuid.qrcode.png -resize 400x400 uuid.qrcode.png

# display contents of QR code, in this case, the UUID
zbarimg -q --nodbus --raw uuid.qrcode.png
cat uuid.txt
```

## Example 2
```bash
## encode YAML in qrcode
cat <<eof | tee id.yaml | qrencode --type=PNG --level=H --output id.qrcode.png
uuid: $( uuidgen )
name: Foo Bar
empid: p123456
email: foo.bar@example.com
eof

# display contents of QR code, in this case, YAML
zbarimg -q --nodbus --raw id.qrcode.png
cat id.yaml

# compare checksums
{ 
zbarimg -q --nodbus --raw id.qrcode.png | sed '$d' | md5sum
cat id.yaml | md5sum
} | uniq -c
```

## Exit session and kill container
```bash
exit
docker stop qrgen

```

## Build the image
```bash
docker build --tag rwcitek/barcode-gen docker/
```

## Example in-line use case: formatting SSNs for input into tax prep software
This takes an SSN, formats it into key strokes, and generates a QR code.
The QR code can then be scanned as keyboard input for tax prep software.
```bash
ssn.qr.code () {
  # takes an SSN as input, formats it into tab separated parts, twice, and then generates QR code
  ssn=${1:-123456789}
  (
  cd $( mktemp -d /tmp/ssn.${ssn}.XXXXXX ) &&
  {
    echo -n ${ssn} | sed -re 's/(...)(..)(....)/\1 \2 \3\t/'
    echo -n ${ssn} | sed -re 's/(...)(..)(....)/\1 \2 \3\n/'
  } | tee ssn.txt | 
  docker run --rm -i rwcitek/barcode-gen \
    qrencode --type=PNG --level=H -o - > ssn.qrcode.png
  )
}
```

## Using pandoc and tidy
Formatting the README.md as HTML
```bash
cat README.md |
docker run -i rwcitek/barcode-gen pandoc --standalone -w html --metadata pagetitle=" " |
docker run -i rwcitek/barcode-gen tidy -ashtml -i -w 0 -u -q > README.html
```
Creating an HTML file that contains the QR image
```bash
{ cat << eof
This is the data in the QR code.
$( echo '```bash' )
$( cat ssn.txt )
$( echo '```')
This is the QR code for entering the SSN
![ssn.qrcode.png](ssn.qrcode.png)
eof
} |
docker run -i rwcitek/barcode-gen pandoc --standalone -w html --metadata pagetitle=" " |
docker run -i rwcitek/barcode-gen tidy -ashtml -i -w 0 -u -q > ssn.qr.html
```
