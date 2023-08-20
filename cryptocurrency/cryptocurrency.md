# Monero (xmr)
[Monero CLI Wallet](https://www.getmonero.org/downloads/#cli)

##  monero-wallet-cli


***

# Miners
## xmrig

[xmrig](https://github.com/xmrig/xmrig)
[xmrig fo moneroocean pool](https://github.com/MoneroOcean/xmrig)

Установка из исходников:

Зависимости:

+ base-devel
+ cmake
+ hwloc
+ libmicrohttpd 

1) Скачаем репозитарий
   
```
git clone https://github.com/xmrig/xmrig
```

2) Отключим минимальное вознограждение, правим файл *xmrig/src/donate.h*

```
constexpr const int kMinimumDonateLevel = 0;
```

3) Соберем программу

```
mkdir build && cd $_
cmake ..
make
```

xmrig -a rx/0 -o  gulf.moneroocean.stream:10128 -u ayfaar -p x --donate-level 0
