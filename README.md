# Generating QR codes within Docker

Initial notes.  Needs some cleanup.

```
docker run --rm --name qrgen ubuntu:20.04 sleep inf & sleep 1

docker exec -i qrgen /bin/bash << 'eof'
  apt-get update &&
  apt-get install -y qrencode uuid-runtime file zbar-tools imagemagick
eof

# create image
docker container commit qrgen qrgen:latest
docker tag qrgen:latest rwcitek/barcode-gen

# exec an interactive session
docker run --rm -it qrgen /bin/bash

# Example 1
# generate UUID and qrcode
uuidgen | tee uuid.txt | qrencode --type=PNG --level=H --output uuid.qrcode.png

# resize png to reasonable size
convert uuid.qrcode.png -resize 400x400 uuid.qrcode.png

# display contents of QR code, in this case, the UUID
zbarimg -q --nodbus --raw uuid.qrcode.png
cat uuid.txt


# Example 2
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


# exit session and kill container
exit
docker stop qrgen

```

## Example use case: formatting SSNs for input into tax prep software
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



