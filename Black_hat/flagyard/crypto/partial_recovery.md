# partial_recovery



**``output.txt``**

```
N = 0xa9f20f8f48075f4abdb1fa55732929e7d70b9ac089abf54ef5896e1e6e3228447be14f9fac2c11a943f4924361d8ff18e19e6314f12662de77480dfe17cfd0479453ba96aee5dc5b12db822b0507d4a49e1d132e062748fa673ac39c2c22cfa38a1f710de2e20c23eb679a70734028851f0a00c522949072d0dae9712f507a13dad2339863fc6c3c3af613995b05fdefb9508ce35434595caea355e331c22b9914dcbe4407ec7f16d2c8fd04465778ed628012f444642a28c85c68e72438c8a7aa0d0079a7eab54faa390fa4c840a45ded2971dd63c444b54063cdfe177fde1cc8cf90fe6d81df297e519f9abd349dfea238b008ba486b4015dcc32e24f8cc03
e = 0xa4ff
c = 0xf6a7384a558ec0dfe3ef1739a7dba5d301c4b1a11774d48c9d20bee85b052f03a4a86b799cc8e99c236ca1126dd5c8b20eb3261691498aef0472569fca6727b8d1d35aa41ae26639f555175d50bcc9c1c64c0fe74a6f7b33226d944d5673e85a2185a9c9d8807bbbde6e70d564b7a29e1d6c6639c028d78f0ab5f57f10867a07f4280ec22b346169cb572e486689b16423ca43f0c5bdf33e0ea093cb54d75d4a28177c6f8ac85390dbc2774589e213456beab117f3f2d08446badb56831b5fe0943d188006803b75726a0b0c0ff3f12ae94164175885457e9be3b09533df55ede2337257e73a602b7922a08b0774705d4dd67e9a1131fbc0b2ae1d38554a245
dp = 0x6d97285825b1ca3a329159ce9ad24cd12766ea83ff50ad3af807ef023bca7a99c03d71a7e819486e15284285c7f962d4e3ae0a068c05e66b2721bb810b763a5d
dq = 0xfb1e95fe13737b13404783341dd3adb1c1a6bc11de2993e3db8d7b67274724c67bdf9d6988b1b9065f4427fcb3e523cec5baad41d005026c7b70ee4908c9a4d7
```



**``server.py``**

```python
from Crypto.Util.number import getPrime, GCD, bytes_to_long
from random import randint


FLAG = str(os.getenv('DYN_FLAG')).encode()
class Crypto:
    def __init__(self, bits):
        self.bits = bits
        self.alpha = 16
        self.delta = 1/4
        self.known = int(self.bits*self.delta)
        print(self.known)
        
    
    def keygen(self):
        while True:
            p, q = [getPrime(self.bits//2) for _ in '__']
            self.e = getPrime(int(self.alpha))
            φ = (p-1)*(q-1)
            try:
                dp = pow(self.e, -1, p-1)
                dq = pow(self.e, -1, q-1)
                self.n = p*q
                break
            except:
                pass

        return (self.n, self.e), (dp, dq)

    def encrypt(self, m):
        return pow(m, self.e, self.n)

rsa = Crypto(2048)
_, (dp, dq) = rsa.keygen()

m = bytes_to_long(FLAG)
c = rsa.encrypt(m)

with open('output.txt', 'w') as f:
    f.write(f'N = 0x{rsa.n:x}\n')
    f.write(f'e = 0x{rsa.e:x}\n')
    f.write(f'c = 0x{c:x}\n')
    print(dp)
    print(dp.bit_length())
    f.write(f'dp = 0x{(dp % 2**(512)):x}\n')
    f.write(f'dq = 0x{(dq % 2**(512)):x}\n')
  #  f.write(f'TWO_POWER = 0x{( 2**(256))):x}\n')
```




-----

```

```




















