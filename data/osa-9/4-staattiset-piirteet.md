---
path: '/osa-9/4-staattiset-piirteet'
title: 'Staattiset piirteet'
hidden: false
---

<text-box variant='learningObjectives' name='Oppimistavoitteet'>

Tämän osion jälkeen:

- Ymmärrät käsitteet staattinen metodi ja luokkamuuttuja
- Tiedät miten staattiset piirteet eroavat olioiden piirteistä
- Osaat lisätä staattisia piirteitä omiin luokkiin

</text-box>

Olio-ohjelmoinnissa puhutaan _piirteistä_. Näillä tarkoitetaan olion ominaisuuksia, luokan sisälle kirjoitettuja aliohjelmia ja siinä määriteltyjä muuttujia.

Tähän mennessä olemme käsitelleen _olioiden piirteitä_ - eli oliometodeita ja attribuutteja. Olio-ohjelmointiin kuuluvat kuitenkin myös _luokan piirteet_, joita kutsutaan usein myös _staattisiksi piirteiksi_. Myös käsitettä _luokkamuuttuja_ käytetään.

## Luokkamuuttujat

Niin kuin on aikaisemmin opittu, jokaisella oliolla on omat itsenäiset arvonsa attribuuteille. Attribuuttien lisäksi luokassa voidaan määritellä _luokkamuuttujia_. Luokkamuuttujalla tarkoitetaan muuttujaa, jota käytetään luokan kautta - ei luokasta muodostettujen olioiden kautta. Luokkamuuttujalla on yksi yhteinen arvo riippumatta siitä, kuinka monta oliota luokasta muodostetaan.

Luokkamuuttujan määrittely eroaa attribuutista siinä, että se määritellään ilman `self`-aluketta. Jos luokkamuuttujaa halutaan käyttää koko luokassa (ja mahdollisesti luokan ulkopuoleltakin), se tulee määritellä metodien ulkopuolella.

```python

class Korkotili:
    yleiskorko = 0.03

    def __init__(self, tilinumero: str, saldo: float, korko: float):
        self.__tilinumero = tilinumero
        self.__saldo = saldo
        self.__korko = korko

    def lisaa_korko(self):
        # korko on yleiskorko + tilin korko
        korko_yhteensa = Korkotili.yleiskorko + self.__korko
        saldo += saldo * korko_yhteensa

    # havainnointimetodit
    @property
    def saldo(self):
        return self.__saldo


```

Koska yleiskorko on määritelty luokassa eikä metodin sisällä, eikä sen alustuksessa ole käytetty `self`-aluketta, se on luokkamuuttuja.

Luokkamuttujaan viitataan _luokan nimen_ avulla, esimerkiksi:

```python

if __name__ == "__main__":
    # Yleiskorko on olioista riippumaton
    print("Yleiskorko on", Korkotili.yleiskorko)

    tili = Korkotili("12345", 1000, 0.05)
    # Lisätään kokonaiskorko saldoon
    tili.lisaa_korko()
    print(tili.saldo)

```

<sample-output>

Yleiskorko on 0.03
1080.0

</sample-output>

Luokkamuuttujiin viitataan siis _luokan nimen avulla_, esimerkiksi `Korkotili.yleiskorko`, ja oliomuuttujiin eli attribuutteihin olion nimen avulla `tili.saldo`. Oliomuuttujiin voi luonnollisesti viitata vasta kun luokasta on muodostettu olio.

Luokkamuuttujaa on kätevä käyttää, kun halutaan arvoja jotka on jaettu kaikkien olioiden kesken. Edellisessä esimerkissä oletetaan, että kaikilla pankkitileillä on sama yleiskorkoprosentti, joiden lisäksi tilille voidaan erikseen määrittää oma korkoprosenttinsa. Yleiskorkokin voi muuttua, mutta muutos vaikuttaa kaikkiin luokasta muodostettuihin olioihin:

```python

class Korkotili:
    yleiskorko = 0.03

    def __init__(self, tilinumero: str, saldo: float, korko: float):
        self.__tilinumero = tilinumero
        self.__saldo = saldo
        self.__korko = korko
        abc = 123

    def lisaa_korko(self):
        # korko on yleiskorko + tilin korko
        korko_yhteensa = Korkotili.yleiskorko + self.__korko
        self.__saldo += self.__saldo * korko_yhteensa

    # havainnointimetodit
    @property
    def saldo(self):
        return self.__saldo

    @property
    def kokonaiskorko(self):
        return self.__korko + Korkotili.yleiskorko

if __name__ == "__main__":
    tili1 = Korkotili("12345", 100, 0.03)
    tili2 = Korkotili("54321", 200, 0.06)

    print("Yleiskorko:", Korkotili.yleiskorko)
    print(tili1.kokonaiskorko)
    print(tili2.kokonaiskorko)

    # Nostetaan yleiskorko 10:en prosenttiin
    Korkotili.yleiskorko = 0.10

    print("Yleiskorko:", Korkotili.yleiskorko)
    print(tili1.kokonaiskorko)
    print(tili2.kokonaiskorko)


```

<sample-output>

Yleiskorko: 0.03
0.06
0.09
Yleiskorko: 0.1
0.13
0.16

</sample-output>

Kun yleiskorko nousee, kaikkien luokasta määriteltyjen tilien kokonaiskorko nousee. Huomaa, että kokonaiskorko on määritelty havainnointimetodiksi, vaikkei vastaavaa attribuuttia olekaan suoraan määritelty - sen sijaan arvoksi lasketaan tilin koron ja yleiskoron summa.

Tarkastellaan vielä toista esimerkkiä. Luokassa `Puhelinnumero` on maatunnukset (esimerkissä toki vain muutama tunnus) tallennettu sanakirjaan. Lista maatunnuksista on yhteinen kaikile luokasta luoduille puhelinnumero-olioille, koska maatunnus saman maan puhelinnumeroille on luonnollisesti aina sama.

```python
class Puhelinnumero:
    maatunnukset = {"suomi": "+358", "ruotsi": "+46", "yhdysvallat": "+1"}

    def __init__(self, nimi: str, puhelinnumero: str, maa: str):
        self.__nimi = nimi
        self.__puhelinnumero = puhelinnumero
        self.__maa = maa

    # Havainnointimetodissa yhdistetään maatunnus ja puhelinnumero
    @property
    def puhelinnumero(self):
        # Puhelinnumerosta jää etunolla pois, kun maatunnus lisätään alkuun
        return Puhelinnumero.maatunnukset[self.__maa] + " " + self.__puhelinnumero[1:]


# Testataan
if __name__ == "__main__":
    paulan_nro = Puhelinnumero("Paula Pythonen", "050 1234 567", "suomi")
    print(paulan_nro.puhelinnumero)
```

<sample-output>

+358 50 1234 567

</sample-output>

Kun puhelinnumero-olio luodaan, tallennetaan nimen ja numeron lisäksi maa. Haettaessa numero havainnointimetodilla, haetaan numeron eteen maatunnus luokkamuttujasta olion attribuuttiin tallennetun maatiedon avulla.

Esimerkkiluokka on toiminnallisuudeltaan kovin vajavainen, katsotaan vielä miltä näyttäisi parempi toteutus, josta löytyy muun muassa havainnointi- ja asetusmetodit eri attribuuteille:

```python

class Puhelinnumero:
    maatunnukset = {"suomi": "+358", "ruotsi": "+46", "yhdysvallat": "+1"}

    def __init__(self, nimi: str, puhelinnumero: str, maa: str):
        self.__nimi = nimi
        # Tämä kutsuu metodia puhelinnumero.setter
        self.puhelinnumero = puhelinnumero
        # ..ja tämä metodia maa.setter
        self.maa = maa

    # Havainnointimetodissa yhdistetään maatunnus ja puhelinnumero
    @property
    def puhelinnumero(self):
        # Puhelinnumerosta jää etunolla pois, kun maatunnus lisätään alkuun
        return Puhelinnumero.maatunnukset[self.__maa] + " " + self.__puhelinnumero[1:]

    @puhelinnumero.setter
    def puhelinnumero(self, numero):
        # Testataan, ettei puhelinnumerossa ole kuin lukuja
        for n in numero:
            if n not in "1234567890 ":
                raise ValueError("Puhelinnumero saa sisältää vain lukuja ja välilyöntejä")
        self.__puhelinnumero = numero

    # Pelkkä puhelinnumero ilman maatunnusta
    @property
    def paikallinen_numero(self):
        return self.__puhelinnumero

    @property
    def maa(self):
        return self.__maa

    @maa.setter
    def maa(self, maa):
        # testataan, että maa löytyy maatunnusten listasta
        if maa not in Puhelinnumero.maatunnukset:
            raise ValueError("Annettua maata ei löydy listalta.")
        self.__maa = maa

    @property
    def nimi(self):
        return self.__nimi

    @nimi.setter
    def nimi(self, nimi):
        self.__nimi = nimi

    # Pelkkä puhelinnumero ilman maatunnusta
    @property
    def paikallinen_numero(self):
        return self.__puhelinnumero

    def __repr__(self):
        return f"Puhelinnumero - nimi: {self.__nimi}, puhelinnumero: {self.__puhelinnumero}, maa: {self.__maa}"


# Testataan
if __name__ == "__main__":
    pnro = Puhelinnumero("Pertti Python", "040 111 1111", "ruotsi")
    print(pnro)
    print(pnro.puhelinnumero)
    print(pnro.paikallinen_numero)

```

<sample-output>

Puhelinnumero - nimi: Pertti Python, puhelinnumero: 040 111 1111, maa: ruotsi
+46 40 111 1111
040 111 1111

</sample-output>

## Staattiset metodit

Myös luokan metodit voivat olla staattisia. Tällaista metodia nimitetään joskus myös luokkametodiksi.

Periaate on samanlainen kuin muuttujienkin kohdalla: staattista metodia ei ole sidottu mihinkään luokasta muodostettuun olioon, vaan se on nimensä mukaisesti luokan metodi. Niinpä staattista metodia (eli luokkametodia) voi kutsua ilman että luokasta muodostetaan oliota.

Staattiset metodit ovatkin yleensä "työkalumetodeja", jotka liittyvät aiheensa puolesta jotenkin luokkaan, mutta joita on tarkoituksenmukaista kutsua ilman että luokasta on pakko muodostaa oliota. Staattiset metodit kirjoitetaan yleensä julkisina, jolloin niitä voidaan kutsua sekä luokan ulkopuolelta että luokan (ja siitä muodostettujen olioiden) sisältä.

Luokkametodi merkitään annotaatiolla `@classmethod`, ja sen ensimmäinen (tai ainoa) parametri on aina `cls`. Tunnistetta `cls` käytetään samaan tapaan kuin tunnistetta `self`, mutta erotuksena on, että `cls` viittaa nykyiseen luokkaan (ja `self` tietysti nykyiseen olioon). Kummallekaan parametrille ei anneta kutsuessa arvoa, vaan Python tekee sen automaattisesti.

Esimerkiksi luokassa `Rekisteriote` voisi olla staattinen metodi, jolla voidaan tarkistaa onko annettu rekisteritunnus oikeamuotoinen. Metodi on staattinen, jotta tunnuksen voi tarkastaa ilman että luodaan uutta oliota luokasta:

```python

class Rekisteriote:

    def __init__(self, omistaja: str, merkki: str, vuosi: int, rekisteritunnus: str):
        self.__omistaja = omistaja
        self.__merkki = merkki
        self.__vuosi = vuosi

        # Kutsutaan metodia rekisteritunnus.setter
        self.rekisteritunnus = rekisteritunnus

    @property
    def rekisteritunnus(self):
        return self.__rekisteritunnus

    @rekisteritunnus.setter
    def rekisteritunnus(self, tunnus):
        if Rekisteriote.rekisteritunnus_kelpaa(tunnus):
            self.__rekisteritunnus = tunnus
        else:
            raise ValueError("Rekisteritunnus ei kelpaa")

    # Luokkametodi tunnuksen validoimiseksi
    @classmethod
    def rekisteritunnus_kelpaa(cls, tunnus: str):
        if len(tunnus) < 3:
            return False

        if "-" not in tunnus:
            return False

        # Tarkastellaan alku- ja loppuosaa erikseen
        alku, loppu = tunnus.split("-")

        # alkuosassa saa olla vain kirjaimia...
        for merkki in alku:
            if merkki.lower() not in "abcdefghijklmnopqrstuvwxyzåäö":
                return False

        # ...ja loppuosassa vain numeroita:
        for merkki in loppu:
            if merkki not in "1234567890":
                return False

        return True

# Testataan
if __name__ == "__main__":
    ote = Rekisteriote("Arto Autoilija", "Volvo", "1992", "abc-123")

    # Metodia voi kutsua myös ilman oliota
    if Rekisteriote.rekisteritunnus_kelpaa("xxx-999"):
        print("Tämä on validi tunnus!")

```

<sample-output>

Tämä on validi tunnus!

</sample-output>

Rekisteriotteen oikeellisuuden voi tarkistaa kutsumalla metodia (esimerkiksi `Rekisteriote.rekisteritunnus_kelpaa("xyz-789"))`) ilman että muodostaa luokasta oliota. Samaa metodia kutsutaan myös uutta oliota muodostaessa luokan konstruktorista - huomaa kuitenkin, että myös tässä kutsussa viitataan metodiin luokan nimen avulla - ei `self`-tunnisteella!
