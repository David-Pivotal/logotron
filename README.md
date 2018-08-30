# logotron

First "tar -xzvf logotron.tgz", then push it with the command "cf push -m 128M -p bin/linux/ -b binary_buildpack -c './logotron' logotron"

 if you want to test with larger log sizes like 100k, you can just bump up the padding size like this.
curl logotron.apps.deschutes.mini.pez.pivotal.io?pad=102400\&code=0x1F428
