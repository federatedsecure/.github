# About this project

**Federated Secure Computing** enables multiple data owners to collaborate without sharing their proprietary data.

- Federated: connects their different servers on-premises and in the cloud
- Secure: encrypts all communication in their private peer-to-peer network
- Computing: acts as a middleware to let them run joint analysis on their data

The motivation, architecture, design, and implementation choices are explained in a detailed 25-page technical whitepaper. The whitepaper has been published in a peer-reviewed open access journal. [Read it here.](https://github.com/federatedsecure/whitepaper)

For non-technical background information, see the project's landing page at [www.federatedsecure.com](https://www.federatedsecure.com).

Support is available at [federatedsecurecomputing@lrz.uni-muenchen.de](mailto:federatedsecurecomputing@lrz.uni-muenchen.de).

The project is hosted by [LMU Munich](https://www.lmu.de/en/index.html) and funded by [Stifterverband](https://www.stifterverband.org/english).

# Getting started

In this simple tutorial, we are going to compute a secure sum between three parties.

Python 3.x is all you need to follow along.

## Server setup

In the Federated Secure Computing architecture, every party runs their own server. The server holds the private data of each party. Computations happen in an encrypted peer-to-peer network between the servers. Only the result of the computation is then revealed to all servers.

In the following example, we are running three servers on the same machine. In reality, they could be all over the world and on very different types of machines.

Let us begin by installing the base server and SIMON, a propaedeutic protocol for 'SImple Multiparty computatiON', so we can actually do some secure calculations:

```
pip install federatedsecure-server
pip install federatedsecure-simon
```

Federated Secure Computing is provided through an OpenAPI 3.0 definition. You can easily generate server stubs for your favourite webserver. We premade a Connexion/Flask app served by Uvicorn:

```
pip install connexion[flask,uvicorn,swagger-ui]
git clone https://github.com/federatedsecure/webserver-connexion
```

For this example, we are running three servers on localhost on ports 55501 through 55503. Feel free to use any other ports:

```
cd webserver-connexion/src
python __main__.py --port=55501 &
python __main__.py --port=55502 &
python __main__.py --port=55503 &
```

You may want to check if the servers are running by browsing to [http://127.0.0.1:55501/representations](http://127.0.0.1:55501/representations).

Done! Your system is now running a functional Federated Secure Computing cluster.

## Client setup

Federated Secure Computing works with clients in any programming language. For the following, we stick to Python. Install the Federated Secure Client wrapper with

```
pip install federatedsecure-client
```

Now copy the following client code to `secure_sum.py`:

``` python
import sys
import federatedsecure.client

# The three servers we started form a peer-to-peer network. Their adresses and ports need to be known to each other:
SHARED_NODES = ['http://127.0.0.1:55501', 'http://127.0.0.1:55502', 'http://127.0.0.1:55503']

# every calculation in Federated Secure Computation is identified by a unique identifier. This UUID is shared by all three servers:
SHARED_UUID = "387a7282-c380-44c9-aede-08da7e931931"

if __name__ == "__main__":

    MY_INDEX = int(sys.argv[1])  # first command line argument identifies the node and must be 0, 1, or 2
    MY_NODE = SHARED_NODES[MY_INDEX]
    MY_NETWORK = {'nodes': SHARED_NODES, 'uuid': SHARED_UUID, 'myself': MY_INDEX}
    MY_SECRET = int(sys.argv[2])  # second command line argument is the secret input

    api = federatedsecure.client.Api(MY_NODE)  # connect to the server
    microservice = api.create(protocol="Simon")  # request the SImple Mulitparty computatiON protocol
    result = microservice.compute(microprotocol="SecureSum", data=MY_SECRET, network=MY_NETWORK)  # and do the calculation
    print(api.download(result))  # receive the result of the computation.
```

That is all.

We now let the three clients run in parallel:

```
python secure_sum.py 0 19 &
python secure_sum.py 1 5 &
python secure_sum.py 2 61
```

The result reads:

```
{
  'inputs': 3,
  'result':
  {
    'samples': 3,
    'sum': 85.0
  }
}
```

Did it compute? Congratulations, you have successfully computed a secure sum.

# Dashboard

<table>
 <thead>
  <tr>
   <td><b>repository</b></td>
   <td><b>license</b></td>
   <td><b>CodeQL</b></td>
   <td><b>rating</b></td>
   <td><b>issues</b></td>
  </tr>
  <tr>
   <td><a href="https://github.com/federatedsecure/api">api</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/api" /></td>
   <td></td>
   <td><img alt="Swagger" src="https://img.shields.io/swagger/valid/3.0?specUrl=https%3A%2F%2Fraw.githubusercontent.com%2Ffederatedsecure%2Fapi%2Fmain%2Fopenapi.yaml" /></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/api" /></td>
  </tr>
  <tr>
   <td><a href="https://github.com/federatedsecure/client-javascript">client-javascript</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/client-javascript" /></td>
   <td><img alt="CodeQL" src="https://github.com/federatedsecure/client-javascript/workflows/CodeQL/badge.svg" /></td>
   <td></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/client-javascript" /></td>
  </tr>
  <tr>
   <td><a href="https://github.com/federatedsecure/client-python">client-python</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/client-python" /></td>
   <td><img alt="CodeQL" src="https://github.com/federatedsecure/client-python/workflows/CodeQL/badge.svg" /></td>
   <td><img alt="Pylint" src="https://raw.githubusercontent.com/federatedsecure/client-python/main/.github/badges/pylint.svg" /></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/client-python" /></td>
  </tr>
   <td><a href="https://github.com/federatedsecure/client-r">client-r</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/client-r" /></td>
   <td></td>
   <td></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/client-r" /></td>
  </tr>
  <tr>
   <td><a href="https://github.com/federatedsecure/server">server</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/server" /></td>
   <td><img alt="CodeQL" src="https://github.com/federatedsecure/server/workflows/CodeQL/badge.svg" /></td>
   <td><img alt="Pylint" src="https://raw.githubusercontent.com/federatedsecure/server/main/.github/badges/pylint.svg" /></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/server" /></td>
  </tr>
  <tr>
   <td><a href="https://github.com/federatedsecure/service-simon">service-simon</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/service-simon" /></td>
   <td><img alt="CodeQL" src="https://github.com/federatedsecure/service-simon/workflows/CodeQL/badge.svg" /></td>
   <td><img alt="Pylint" src="https://raw.githubusercontent.com/federatedsecure/service-simon/main/.github/badges/pylint.svg" /></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/service-simon" /></td>
  </tr>
  <tr>
   <td><a href="https://github.com/federatedsecure/webserver-connexion">webserver-connexion</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/webserver-connexion" /></td>
   <td><img alt="CodeQL" src="https://github.com/federatedsecure/webserver-connexion/workflows/CodeQL/badge.svg" /></td>
   <td><img alt="Pylint" src="https://raw.githubusercontent.com/federatedsecure/webserver-connexion/main/.github/badges/pylint.svg" /></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/webserver-connexion" /></td>
  </tr>
  <tr>
   <td><a href="https://github.com/federatedsecure/webserver-django">webserver-django</a></td>
   <td><img alt="license" src="https://img.shields.io/github/license/federatedsecure/webserver-django" /></td>
   <td><img alt="CodeQL" src="https://github.com/federatedsecure/webserver-django/workflows/CodeQL/badge.svg" /></td>
   <td><img alt="Pylint" src="https://raw.githubusercontent.com/federatedsecure/webserver-django/main/.github/badges/pylint.svg" /></td>
   <td><img alt="issues" src="https://img.shields.io/github/issues/federatedsecure/webserver-django" /></td>
  </tr>
 </thead>
</table>
