**Charybdis 3*6 downgle**

![image](https://github.com/user-attachments/assets/0ff7d0b6-d020-493d-9c3a-81cbd71db521)

This repo for charybdis 3*6 with dongle.
Work on the project continues
![my_keymap](https://github.com/user-attachments/assets/db53c185-02c7-47e4-a2d5-2270d5ada041)

Для работы сенсора 3610 на плате elite-c holder 2.1
Итак, если вы стали обладателем платы-переходника elite-c holder 2.1 и сенсора 3610, и Вы хотите заставить работать трекбол, эта статья для вас.
Начать стоит с того, что нужно распаять правильно пины.
На плате сенсора у Вас их будет 6 (GND\MO\SCLK\SDIO\NCS\VCC). Эти пины мы будем паять на плату переходник elite-c holder 2.1.
Распиновка следующая
| GND | GND |
|---|---|
| VCC | 5V |
|---|---|
| MO | MISO |
|---|---|
| SCLK | SCLK |
|---|---|
| SDIO | MOSI |
|---|---|
| NCS | 101 pin on nice!nano(nrf52840) |
|---|---|
Далее переходим к редактированию прошивки. В моес случае это charybdis_3610.dtsi
Первый скриншот это то что нужно вставить, второй - значение по умолчанию, которое будет у Вас после форка проекта.
```
&pinctrl {
    spi0_default: spi0_default {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 1, 13)>,
                <NRF_PSEL(SPIM_MOSI, 0, 10)>,
                <NRF_PSEL(SPIM_MISO, 0, 10)>;
        };
    };

    spi0_sleep: spi0_sleep {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 1, 13)>,
                <NRF_PSEL(SPIM_MOSI, 0, 10)>,
                <NRF_PSEL(SPIM_MISO, 0, 10)>;
            low-power-enable;
        };
    };
};
```

```
&pinctrl {
    spi0_default: spi0_default {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 8)>,
                <NRF_PSEL(SPIM_MOSI, 0, 17)>,
                <NRF_PSEL(SPIM_MISO, 0, 17)>;
        };
    };

    spi0_sleep: spi0_sleep {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 8)>,
                <NRF_PSEL(SPIM_MOSI, 0, 17)>,
                <NRF_PSEL(SPIM_MISO, 0, 17)>;
            low-power-enable;
        };
    };
};
```

И добавить блок кода в низ файла
```
/ {
  trackball_listener {
    compatible = "zmk,input-listener";
    device = <&trackball>;

  };
};
```

Так же в конфигурации требуется добавить поддержку трекбола. Но если у Вас форк проекта с трекболом, то там уже все будет добавлено.
В файле charybdis_dongle.conf нужно добавить 
CONFIG_PMW3610=y
CONFIG_SPI=y

А так же в файл west.yml
```
    - name: badjeff
      url-base: https://github.com/badjeff

    - name: zmk-pmw3610-driver
      remote: badjeff
      revision: main
    - name: zmk-split-peripheral-input-relay
      remote: badjeff
      revision: main
    - name: zmk-input-behavior-listener
      remote: badjeff
      revision: main
```
      
