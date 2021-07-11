# 闲题杂记

<!--more-->

[toc]

## gkctf2021 6-25

太菜了,只做了两个题,时间还不够了呜呜


![gkctf](https://dm2302files.storage.live.com/y4mvXXgHz2RHVVQz0ulf35x2b2Txeabor6e5MNxHal27mW8iV3bZvJjXTd7LVryDMrR69bDzZMw71EP3VZWDam3Z_Zxt7b0IEzvsGeEIq-WBfVPHPJI_dE4tsT_1uualuy_CROcC24VnS86fjiGjrVG9FgSCrFE-9DTtC1bXxzR4R7qHR5qSME5GIIpyP8t9bTO?width=1880&height=944&cropmode=none)

### pythonrandom通杀脚本

```python
from randcrack import RandCrack
rc = RandCrack()
for i in data:
    r = bin(int(i))[2:].zfill(64)
    r1 = r[:32]
    r2 = r[32:]
    rc.submit(int(r2, 2))
    rc.submit(int(r1, 2))

tmp = rc.predict_getrandbits(64)
```

### rrrrsa

小数论题

注意mod n和mod q之间的关系

```python
from libnum import *
from Crypto.Util.number import *
from icecream import *
e = 65537
c=13492392717469817866883431475453770951837476241371989714683737558395769731416522300851917887957945766132864151382877462142018129852703437240533684604508379950293643294877725773675505912622208813435625177696614781601216465807569201380151669942605208425645258372134465547452376467465833013387018542999562042758
n1=75003557379080252219517825998990183226659117019770735080523409561757225883651040882547519748107588719498261922816865626714101556207649929655822889945870341168644508079317582220034374613066751916750036253423990673764234066999306874078424803774652754587494762629397701664706287999727238636073466137405374927829

c1=68111901092027813007099627893896838517426971082877204047110404787823279211508183783468891474661365139933325981191524511345219830693064573462115529345012970089065201176142417462299650761299758078141504126185921304526414911455395289228444974516503526507906721378965227166653195076209418852399008741560796631569
cc1=23552090716381769484990784116875558895715552896983313406764042416318710076256166472426553520240265023978449945974218435787929202289208329156594838420190890104226497263852461928474756025539394996288951828172126419569993301524866753797584032740426259804002564701319538183190684075289055345581960776903740881951
cc2=52723229698530767897979433914470831153268827008372307239630387100752226850798023362444499211944996778363894528759290565718266340188582253307004810850030833752132728256929572703630431232622151200855160886614350000115704689605102500273815157636476901150408355565958834764444192860513855376978491299658773170270
# hint1 = pow(2020 * p1 + q1, 202020, n1)
# hint2 = pow(2021 * p1 + 212121, q1, n1)
a = 2020
e1 = 202020
e2 = 212121
tmp = ((cc2-e2)*a*invmod(a+1,n1))%n1
tmp = pow(tmp,e1,n1)-cc1%n1
q1 = gcd(tmp,n1)
p1 = n1//q1
ic(q1,n1%q1)
phi1 = (p1-1)*(q1-1)
d1 = invmod(e,phi1)
P = pow(c1,d1,n1)


a = 2020
e1 = 202020
e2 = 212121
t = e1*e2
n2=114535923043375970380117920548097404729043079895540320742847840364455024050473125998926311644172960176471193602850427607899191810616953021324742137492746159921284982146320175356395325890407704697018412456350862990849606200323084717352630282539156670636025924425865741196506478163922312894384285889848355244489
c2=67054203666901691181215262587447180910225473339143260100831118313521471029889304176235434129632237116993910316978096018724911531011857469325115308802162172965564951703583450817489247675458024801774590728726471567407812572210421642171456850352167810755440990035255967091145950569246426544351461548548423025004
cc1=25590923416756813543880554963887576960707333607377889401033718419301278802157204881039116350321872162118977797069089653428121479486603744700519830597186045931412652681572060953439655868476311798368015878628002547540835719870081007505735499581449077950263721606955524302365518362434928190394924399683131242077
cc2=104100726926923869566862741238876132366916970864374562947844669556403268955625670105641264367038885706425427864941392601593437305258297198111819227915453081797889565662276003122901139755153002219126366611021736066016741562232998047253335141676203376521742965365133597943669838076210444485458296240951668402513
f1 = cc2 *pow(a,e2,n2)*invmod(pow(a+1,e2,n2),n2)%n2
tmp = (pow(f1,e1,n2)-pow(cc1,e2,n2))%n2
q2 = gcd(tmp,n2)
ic(q2,n2%q2)
p2 = n2//q2
phi2 = (p2-1)*(q2-1)
d2 = invmod(e,phi2)
Q = pow(c2,d2,n2)


e = 65537
c=13492392717469817866883431475453770951837476241371989714683737558395769731416522300851917887957945766132864151382877462142018129852703437240533684604508379950293643294877725773675505912622208813435625177696614781601216465807569201380151669942605208425645258372134465547452376467465833013387018542999562042758
p=P
q=Q
n=p*q
phi = (p-1)*(q-1)
d = invmod(e,phi)
m = pow(c,d,n)
print(long_to_bytes(m))
# GKCTF{f64310b5-d5e6-45cb-ae69-c86600cdf8d8}
```

### XOR

```python
from Crypto.Util.number import *
from hashlib import md5

a = getPrime(512)
b = getPrime(512)
c = getPrime(512)
d = getPrime(512)
d1 = int(bin(d)[2:][::-1] , 2)
n1 = a*b
x1 = a^b

n2 = c*d
x2 = c^d1
flag = md5(str(a+b+c+d).encode()).hexdigest()
print("n1 =",n1)
print("x1 =",x1)
print("n2 =",n2)
print("x2 =",x2)
```



这个题基本是靠约束条件对多余情况进行剪枝，捡到运算量在合理范围就可以了

顺序不变时只需要考虑低位，除开异或条件外还计 算 $a*b =n\;mod\;2^{i}-1$ 来对已猜测数据进行低位检测

这是一种模糊的条件，该条件是最终条件的必要条件

---

顺序改变的情况，异或需要考虑高低位交换的情况，每次要四个bit同时运算看是否同时满足 n的高位和低位

乘法条件中，低位由于没有进位，直接判断$a*b =n\;mod\;2^{i}-1$

高位由于又进位，使用高位相同时的必要条件`if n_highbits-temp2 >= 0 and n_highbits-temp2 <=((2<<i+1)-1): `

a = 111

b = 100

a*a = 1 1000 1

b*b = 010000

当然，这依旧是一个十分粗略的必要条件。。。。

正序exp

```python

# 初始化第1位的已知数：0
def getab(n,x,lenth):
    a_list=[0]
    b_list=[0]
    # 这里判断512位应该就够了阿。。。。
    mask = 0
    for i in range(lenth):
        # 取第n位
        mask = 2**(i+1)-1
        xi = (x>>i) & 1
        nextA_list=[]
        nextB_list=[]
        
        for ai in range(2):
            for bi in range(2):
                for j in range(len(a_list)):
                    if (ai^bi == xi):
                        nlow = n & mask
                        axbLow = (((ai<<i)+a_list[j])*((bi<<i)+b_list[j]))&mask
                        if(nlow==axbLow):
                            nextA_list.append((ai<<i)+a_list[j])
                            nextB_list.append((bi<<i)+b_list[j])
        # 
        a_list = nextA_list
        b_list = nextB_list

    for a in a_list:
        if(n%a==0):
            return(a,n//a)

lenth = 512

n = 83876349443792695800858107026041183982320923732817788196403038436907852045968678032744364820591254653790102051548732974272946672219653204468640915315703578520430635535892870037920414827506578157530920987388471203455357776260856432484054297100045972527097719870947170053306375598308878558204734888246779716599
x = 4700741767515367755988979759237706359789790281090690245800324350837677624645184526110027943983952690246679445279368999008839183406301475579349891952257846
a,b = getab(n,x,lenth)
from icecream import *
ic(a,b)
```



倒序exp

```python
def get_cd(n,x,lenth): 
	p_low = [0] 
	q_high = [0]
	q_low = [0] 
	p_high = [0] 
	# maskn = 2
	maskn  = 0
	for i in range(lenth//2): 
		maskn = 2**(i+1)-1
		xi = (x >> i )&1
		n_lowbits = (n & maskn) 
		# 高位判断从lenth-1处开始
		High_index = lenth-1 -i
		XHi = (x >> (High_index))&1 
		n_highbits = (n)>> (High_index) *2
		nextP_l = [] 
		nextQ_l = [] 
		nextP_h =[] 
		nextQ_h =[] 
		
		for j in range(len(p_low)): 
			for pl in range(2): 
				for ql in range(2): 
					for ph in range(2): 
						for qh in range(2): 
							if pl ^ qh == xi and ql ^ ph == XHi:
								PlxQl = (((pl<<i) + p_low[j]) * ((ql<<i) + q_low[j])) & maskn 
								PhxQh = (((ph << (High_index)) + p_high[j]) * ((qh << (High_index)) + q_high[j]))>>(High_index)*2 
								if PlxQl == n_lowbits : 
									# if n_highbits-PhxQh >= 0 and n_highbits-PhxQh <=((2<<i+1)-1)
									# 高n位的差在 2^(i+1)-1以内是 高位相同的必要条件
									if n_highbits-PhxQh >= 0 and n_highbits-PhxQh <=((2<<i+1)-1): 
										nextP_l.append((pl<<i) + p_low[j]) 
										nextQ_l.append((ql<<i) + q_low[j]) 
										nextP_h.append((ph<<(High_index))+p_high[j]) 
										nextQ_h.append((qh<<(High_index))+q_high[j]) 
		p_low = nextP_l 
		q_low = nextQ_l 
		p_high = nextP_h 
		q_high = nextQ_h 
	for a in p_low: 
		for b in p_high: 
			if n %(a+b) ==0: 
				p = a + b 
				q = n//p                                             
				print(p,q)
				break


n2 = 65288148454377101841888871848806704694477906587010755286451216632701868457722848139696036928561888850717442616782583309975714172626476485483361217174514747468099567870640277441004322344671717444306055398513733053054597586090074921540794347615153542286893272415931709396262118416062887003290070001173035587341
x2 =3604386688612320874143532262988384562213659798578583210892143261576908281112223356678900083870327527242238237513170367660043954376063004167228550592110478
lenth = 512
get_cd(n2,x2,lenth)

# ic(p,q)
```

稍微改了一下原p阴间的位运算





## n1ctf2021

> 咕咕咕了好久

### vss

- 难点在随机数预测上面



先使用了一个二维码生成

```python
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=12,
        border=4,
    )
    qr.add_data(FLAG)
    qr.make(fit=True)
    img = qr.make_image(fill_color="black", back_color="white")
```

RGB = 0xffffff 是白色

RGB = 0 是黑色

在填充像素时 

当 pixel !=255 时填充在 2x, 2y 的 color会不一样 

若能获得一大段连续的一样的pixel ，只要通过判断n个连续 color0 / color1的值就可以恢复出随机数MT9937的当前状态，不管是向前还是向后推都可以得到加密图片使用的随机数

```python
        if pixel:
            ...
        else:
            share1.putpixel((2*x, 2*y), color0)
            share1.putpixel((2*x, 2*y+1), color0)
            ...

            share2.putpixel((2*x, 2*y), color1)
            share2.putpixel((2*x, 2*y+1), color1)
            ...
```

exp

```python
from PIL import Image
from randcrack import RandCrack
import random
share = Image.open('./share2.png')
width = share.size[0]//2
res = Image.new('L', (width, width))
bits = ''

# pixel为1填充0
# pixel为0填充1
# 01分别对应的是黑色的填充和白色的背景像素 
# 官p取最后一段连续白色
for idx in range(width*width-624*32, width*width):
    i, j = idx//width, idx % width
    if share.getpixel((2*j, 2*i)) == 255:
        bits += '0'
    else:
        bits += '1'
# 判断像素后
rc = RandCrack()
for i in range(len(bits), 0, -32):
    rc.submit(int(bits[i-32:i], 2))
flipped_coins = [int(bit) for bit in bin(rc.predict_getrandbits(width*width-624*32))[2:].zfill(width*width-624*32)] + list(map(int, bits))

data = []

for idx in range(width*width):
    i, j = idx//width, idx % width
    if share.getpixel((2*j, 2*i)) == 255:
        data.append(0 if flipped_coins[idx] else 255)
    else:
        data.append(255 if flipped_coins[idx] else 0)

res.putdata(data)
res.save('ans.png')
```


