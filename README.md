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

# miscelaneous

To create a typeorm migration:

```console
npx typeorm migration:create -n NAME
```

Display the last error on Unix

```console
echo $?
```

Get the request information in cURL

```console
curl -I <http-address>
```

To change the language of Brother printers (Firmware configuration change):

1. Switch on printer
2. Hold during several seconds the Home button on the printer option display. 
3. Introduce in the numeric pad the code `*2864`
4. Select the menu `74` and press afterwards `Mono start`
5. Introduce the language code (`0066` for Iberian/Italian or `0015` for Spanish depending of the page loaded) and wait until the printer stores the option. 
6. Restart the printer by introducing `99`
