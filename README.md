# recipes
Technical recipes and notes for technical stuff

- k8s
- docker
- linux
- git
- js/ts
- jenkins
- react
- node
- python
- linux
- go
- kafka
- git
- ansible
- vagrant
- packer
- postgres
- .net
- mongo

## Appendix: nodeJS typeorm migrations

To create a typeorm migration:

```console
npx typeorm migration:create -n NAME
```

## Appendix: Display the last error on Unix

```console
echo $?
```

## Appendix: Get the request information in cURL

```console
curl -I <http-address>
```

## Appendix: To change the language of Brother printers (Firmware configuration change):

1. Switch on printer
2. Hold during several seconds the Home button on the printer option display. 
3. Introduce in the numeric pad the code `*2864`
4. Select the menu `74` and press afterwards `Mono start`
5. Introduce the language code (`0066` for Iberian/Italian or `0015` for Spanish depending of the page loaded) and wait until the printer stores the option. 
6. Restart the printer by introducing `99`

## Appendix: ffmpeg links

https://stackoverflow.com/questions/21184014/ffmpeg-converted-mp4-file-does-not-play-in-firefox-and-chrome
https://stackoverflow.com/questions/58506549/videos-which-dont-play-in-firefox-how-to-detect-and-how-to-fix
https://stackoverflow.com/questions/38489231/ffmpegphp-videos-modified-by-ffmpeg-are-not-played-in-firefox

## Appendix: OpenSSL 

To generate a new Certificate Signing Request

```console
openssl req -newkey rsa:2048 -keyout <output-key-file> -out <output-csr-file>
```

To query all information about a Certificate Signing request or an x509 certificate

CSR:

```console
openssl req -in <input-csr-file> -text -noout
```

x509:

```console
openssl x509 -in <input-crt-file> -text -noout
```

To query only the subject of a CSR and x509 certificate

CSR:

```console
openssl req -in <input-csr-file> -subject -noout
```

x509:
```console
openssl x509 -in <input-crt-file> -subject -noout
```

To query only the expiration date of a x509 certificate

```console
openssl x509 -in <input-crt-file> -enddate -noout
```

## Appendix: OpenSSH and connect with RSA private key

On the server host, enable the SSH server (Not enabled by default in Ubuntu clients)

```console
sudo apt update
sudo apt install openssh-server
```

Check status of SSH server

```console
sudo systemctl status ssh
```

Enable SSH port over firewall

```console
sudo ufw allow ssh
```

For connecting over SSH and private key

on client host, generate a RSA key for connecting

```console
ssh-keygen -t rsa -b 4096
```

After generating the keys, copy it to the remote host by using `ssh-copy-id`. Once prompted for the remote host server credentials, the authorized public key will be copied to the list. 

```console
ssh-copy-id <user>@<host>
```

Connect to user at host from the client host: 

```console
ssh abel@ale-laptop2
```

## Appendix: Ghostscript and pdftk

To compress a PDF file using GS (Windows x64)

```console
gswin64 -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook -dNOPAUSE -dQUIET -dBATCH -sOutputFile=<OUTPUT_FILE_NAME>.pdf <INPUT_FILE_NAME>.pdf
```

To concatenate several PDF files

```console
pdftk <file_1.pdf> <file_2.pdf> ... <file_N.pdf> cat output <output_file.pdf>
```
