# Butane smallstep ACME RA

This is a repo for launching an step-ca setup as an ACME RA with a jinja2 template. You can render it easily with [Bupy](https://github.com/quickvm/bupy). Just customize the `butanevars.yaml` file and run Bupy to process it.

```
bupy template smallstep-acme-ra.bu.j2 butanevars.yaml --show
```

Signup for a free smallstep certificates account [here](https://smallstep.com/signup?product=cm).


## Setup RA JWK provision on smallstep.com account

```
mkdir -p $HOME/Documents/step
chmod 0740 $HOME/Documents/step
export STEPPATH=$HOME/Documents/step
step path
step ca bootstrap --context mycontext --ca-url https://myca.myteam.ca.smallstep.com --fingerprint myfingerprint
passwd 35 > myjwkpassword
step ca provisioner add acme-ra --type JWK --create --password-file=myjwkpassword
rm -f myjwkpassword
```

## License

MIT License

Copyright (c) 2022 Smallstep Labs, Inc

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

