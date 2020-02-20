---
title: "Python ile Komut dizayn kalıbı"
date: 2020-02-20T09:37:43+03:00
draft: false
categories: ["Design Pattern", "Python"]
---

## Command Desing Pattern

Bu yazıda **behavioral** kategorisine dahil olan **Command Design Pattern** python dilinde nasıl kullanılır onu anlatacağım.

### Kullanım Amacı

Kuyruk (Queue) sistemlerinde genel olarak kullanılır. Yeni özellikler eklenebilecek sistemlerde kullanılabilir.

### Örnek

Örnek basit olması açısından bir TV sınıfının özelliklerini **Command Design Pattern** ile kullanacağız.

**Adım 1**

- Öncelikle komutlarımızın kalıtım olarak alabilmesi için bir **Command** sınıfı yaratıyoruz.

```python
from abc import ABC, abstractmethod

class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
```

**Adım 2.**

- Üstte oluşturduğumuz sınıftan kalıtım alarak TV sınıfını kontrol edecek sınıfları yazalım

```python
class VolumeUpCommand(Command):
    def __init__(self, tv):
        self._tv = tv

    def execute(self):
        if tv.volume+1 <= tv.MAX_VOLUME:
            tv.volume = tv.volume + 1
	    return tv.volume
        raise BaseException('Volume Error')

class VolumeDownCommand(Command):
    def __init__(self, tv):
        self._tv = tv
    def execute(self):
        if tv.volume - 1 >= 0:
            tv.volume = tv.volume - 1
	    return tv.volume
        raise BaseException('Volume Error')

class MuteTV(Command):
    def __init__(self, tv):
        self._tv = tv
    def execute(self):
        tv.volume = 0
        return True
```

- Evet gerekli kontrollerle beraber TV sınıfımızın volume değişkeniyle ilgili komutlar oluşturduk.

**Adım 3**

- Şimdi ise bu komutları çalıştıracak bir sınıfa ihtiyacımız var

```python
class RemoteController:
    def __init__(self):
        self.__command = None
    def set_command(self, command):
        self.__command = command
    def run(self):
        if isinstance(self.__command, Command):
            return self.__command.execute()
```

- Burada `self.__command` değişkenine hangi sınıfın geldiğinin bir önemi yok. `RemotController` sadece bu sınıfı çalıştırır. İhtiyaç halinde history diye bir list oluşturulup yapılan değişikler orada tutulabilir.

**Adım 4**

- Komutların çalıştıracağı TV sınıfımızı oluşturuyoruz.

```python
class TV:
    MAX_VOLUME = 10
    def __init__(self):
        self.volume = 0
```

**Adım 5**

- Son durumda kodun çalışma şekli bu şekilde olacak.

```python
tv = TV()
volume_down = VolumeDownCommand(tv)
volume_up = VolumeUpCommand(tv)
mute = MuteCommand(tv)
remote_controller = RemoteController()
remote_controller.set_command(volume_up)
remote_controller.run()
print(tv.volume) # 1
remote_controller.set_command(volume_down)
remote_controller.run()
print(tv.volume) # 0
```

### Sonuç

Bu tasarım kalıbı sayesinde yeni bir özellik eklemek istediğimiz zaman yapmamız gereken tek şey `Command` sınıfından
kalıtım alarak bir sınıf üretmek. Ardından `RemoteController` sınıfı bu yeni komutumuzu istediğimiz zaman çalıştırabilecektir. Bu sayede uzun `if(foo): bar()` şeklinde kontroller yapmamıza gerek kalmadı.
