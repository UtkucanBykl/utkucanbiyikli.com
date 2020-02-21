---
title: "Gözlemci Dizayn Kalıbı ve Django"
date: 2020-02-20T13:57:28+03:00
draft: false
toc: false
images:
---## Python ve Gözlemci Dizayn Kalıbı

Merhaba bu yazım **Observer Design Pattern** (Gözlemci Dizayn Kalıbı) hakkında olacak.

#### Nedir ?

**Observer Design Pattern**, bir **sınıfımızda** bir değişiklik olduğu zaman bu sınıfımıza bağlı sınıfları uyarmamızı sağlayan bir yapıdır. En güzel örnek ise **_Django’nun_** da kullandığı **_Signal(Sinyal)’lerdir._** Bu sinyaller sayesinde **_Model_** bir kayıt yapacağı zaman **pre_save, post_save** gibi **sinyaller** gider. Bu sinyallerde **_Observer Design Pattern_** kullanılmıştır. Bu yazıda da sinyallerin basit bir halini kodlayacağız.

- **Adım 1** <br>
  İlk önce yapmamız gereken şey bir **Signal** sınıfı oluşturup, bu sınıfa modelin **Save** methodu çalıştığında çalışacak fonksiyonları tutacak ve çalıştıracak fonksiyonları yazmamız gerekiyor.

```python
class Signal:
    def __init__(self):
        self._receivers = []
    def connect(self, receiver, name=None, sender=None):
        if callable(receiver):
            name = name or receiver.__name__
            receiver_data = {
                'function': receiver,
                'name': name,
                'sender': sender
            }
            self._receivers.append(receiver_data)
        else:
            raise BaseException('Receiver must be callable')

    def disconnect(self, receiver=None, name=None):
        if receiver:
            name = name or receiver.__name__
        for item in self._receivers:
            if item['name'] == name:
                del item
                break

    def send(self, instance):
        for item in self._receivers:
            if isinstance(instance, item['sender']) or item['sender'] is None:
                item['function'].__call__()

pre_save = Signal()
post_save = Signal()
```

Öncelikle ilk sınıfımızı oluştururken ileride **Modelimizin Save fonksiyonu** çalışınca çalışacak fonksiyonları tutan **self.\_receivers** listemizi oluşturuyoruz.

**def connect** ile fonksiyonları listemize ekliyoruz. Burada önemli olan **_receiver_** diye verdiğimiz argümanın bir fonksiyon olması. Bunu da **callable**() ile kontrol ediyoruz. Listemize bu fonksiyonu, özel bir isim verdiysek onu -_yoksa fonksiyon ismini_- ve hangi modelin save fonksiyonu ile tetikleneceği bilgisi için o Model sınıfını argüman olarak veriyoruz. Bu argümanları bir **dictionary** içine atıp listemize ekliyoruz.

**def disconnect** fonksiyonu da ilerde istenmeyen fonksiyonların çıkarılması için yazıyoruz.

**def send** fonksiyonu bir adet **model nesnesi** alıyor ve tek tek **self.\_receivers** listesini gezip, aldığı **nesne** listeye koyduğumuz **dictionarydeki Sender** sınıfından mı türemiş buna bakıyor. Eğer öyleyse listeye koyduğumuz **dictionarydeki function keyini** çağırıp ****call**** ile fonksiyonu çağırıyor.

Ardından **Signal** sınıfından **pre_save** ve **post_save** diye iki adet signal nesne üretiyoruz.

- **Adım 2** <br>
  Basit bir model sınıfı oluşturuyoruz. Bu model sınıfımız bir adet **save()** fonksiyonuna sahip. Bu save fonksiyonu şu an için hiçbir şey yapmıyor. Biz save fonksiyonun en tepesine **pre_save.send(self)** ile önceden eklemiş olduğumuz fonksiyonları çalıştırmasını söylüyoruz. Ardından save işlemlerini yapıp -burada böyle bir kod yok- son olarak **post_save.send(self)** ile kayıt olduktan sonra yapılacak işlemleri çağırıyoruz. -self yollayarak objenin kendisini nesne olarak veriyoruz

```python
from signal import pre_save, post_save

class Model:
    def save(self):
        pre_save.send(self)
        # awesome save lines
        post_save.send(self)
        return obj
```

- **Adım 3** <br>
  İki adet basit **post_save_method** ve **pre_save_method** yazıyoruz.Bu methodlar print ile ekrana çalıştıklarına dair bilgi veriyorlar. Ardından bunları **post_save.connect** ve **pre_save.connect** ile listemize ekliyoruz. Artık ne zaman bir **model.save()** çalışırsa eklediğimiz methodlar da çalışacak.

```python
from signal import pre_save, post_save
from model import Model
def post_save_method():
    print('post save method run')
def pre_save_method():
    print('pre save method run')
post_save.connect(post_save_method, sender=Model)
pre_save.connect(pre_save_method, sender=Model)
model = Model()
model.save()
# çıktılar
# pre save method run
# post save method run
```

---_Önceki Design Pattern Yazım için_
[**Python ve Komut Dizayn Kalıbı**  
](https://medium.com/@utkucanbykl/python-ve-komut-dizayn-kal%C4%B1b%C4%B1-d6027304f00f)
