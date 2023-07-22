# Generating QR codes within Docker

## Example: "Hello, world!"
### Encoding "Hello, world!" into a QR code
```bash
echo 'Hello, world!' |
docker container run --rm -i rwcitek/barcode-gen \
  qrencode --type=PNG --level=H -o - > hello-world.qrcode.png
```
### Decoding "Hello, world!" QR code
```bash
cat hello-world.qrcode.png |
docker container run --rm -i rwcitek/barcode-gen \
  zbarimg -q --nodbus --raw -
```

## Running a background container
Run a service ( daemon ) instance in the background
```bash
docker container run -d --name qrgen rwcitek/barcode-gen sleep inf ; sleep 1
```
Using the background instance to create or decode a QR code similar to the code above.

```bash
# Encoding "Hello, world!" to QR code
echo 'Hello, world!' |
docker container exec -i qrgen \
  qrencode --type=PNG --level=H -o - > hello-world.qrcode.v02.png
```

```bash
# Decoding "Hello, world!" QR code
cat hello-world.qrcode.v02.png |
docker container exec -i qrgen \
  zbarimg -q --nodbus --raw -
```

## An interactive session using the background container
```bash
# exec into it
docker container exec -it qrgen /bin/bash
```

### Example 1
```bash
# generate UUID and qrcode
uuidgen | tee uuid.txt | qrencode --type=PNG --level=H --output uuid.qrcode.png

# resize png to reasonable size
convert uuid.qrcode.png -resize 400x400 uuid.qrcode.png

# display contents of QR code, in this case, the UUID
zbarimg -q --nodbus --raw uuid.qrcode.png
cat uuid.txt
```

### Example 2
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

### Exit interactive session
```bash
exit
```

## Update/enhance instance
You can update/enhance the instance by upgrading or adding additional packages.
For example, you can install the `dmtx-utils` package to read/write data matrix barcodes 
and the `libeif-examples` to convert HEIF images (iPhone) to JPEG or PNG.
```bash
# Update/enhance
docker container exec -i qrgen /bin/bash <<'eof'
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get -y dist-upgrade
  apt-get install -y \
    dmtx-utils \
    libheif-examples \
  ;
eof
```

For example, create and read a Data Matrix bar code encoding "Hello, world!".
```bash
# Create Data Matrix barcode
echo 'Hello, world!' |
docker container exec -i qrgen \
  dmtxwrite -f PNG -o - > hello-world.data-matrix.png
```
```bash
# Read Data Matrix barcode
cat hello-world.data-matrix.png |
docker container exec -i qrgen \
  dmtxread -
```

### Stop and remove container
```bash
docker container stop qrgen
docker container rm qrgen
```

## Build the image
```bash
docker image build --tag rwcitek/barcode-gen docker/
```

## Example in-line use case: formatting SSNs for input into tax prep software
This takes an SSN, formats it into key strokes, and generates a QR code.
The QR code can then be scanned as keyboard input for tax prep software.
```bash
ssn.qr.code () {
  # takes an SSN as input, formats it into tab separated parts, twice, and then generates QR code
  ssn=${@:-123456789}
  ssn=$( <<< "${ssn}" tr -dc [0-9] )
  (
  cd $( mktemp -d /tmp/ssn."${ssn}".XXXXXX ) &&
  {
    echo -n "${ssn}" | sed -re 's/([0-9]{3})([0-9]{2})([0-9]{4})/\1 \2 \3\t/'
    echo -n "${ssn}" | sed -re 's/([0-9]{3})([0-9]{2})([0-9]{4})/\1 \2 \3\n/'
  } | tee ssn.txt | 
  docker container run --rm -i rwcitek/barcode-gen \
    qrencode --type=PNG --level=H -o - > ssn.qrcode.png
  )
}
```

## Using pandoc and tidy
Formatting the README.md as HTML
```bash
cat README.md |
docker container run -i rwcitek/barcode-gen pandoc --standalone -w html --metadata pagetitle=" " |
docker container run -i rwcitek/barcode-gen tidy -ashtml -i -w 0 -u -q > README.html
```
Creating an HTML file that contains the QR image
```bash
{ cat << eof
This is the data in the QR code.
<pre>
$( od -bc ssn.txt )
</pre>
![QR code for SSN](ssn.qrcode.png){ width=200px }

eof
} |
docker container run -i rwcitek/barcode-gen pandoc --standalone -w html --metadata pagetitle=" " |
docker container run -i rwcitek/barcode-gen tidy -ashtml -i -w 0 -u -q > ssn.qr.html
```
