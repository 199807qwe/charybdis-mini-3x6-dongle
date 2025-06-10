**Charybdis 3*6 downgle**

This repo for charybdis 3*6 with dongle.
Work on the project continues

![56cf454f-f5c7-4c36-aa63-ef28df3399b7](https://github.com/user-attachments/assets/c49da18a-4c8f-4ecf-998b-0d296778741f)
![image](https://github.com/user-attachments/assets/e031d0af-3228-49c2-ba74-56f261c80ddb)


Для работы сенсора 3610 на плате elite-c holder 2.1
Итак, если вы стали обладателем платы-переходника elite-c holder 2.1 и сенсора 3610, и Вы хотите заставить работать трекбол, эта статья для вас.
Начать стоит с того, что нужно распаять правильно пины.
На плате сенсора у Вас их будет 6 (GND\MO\SCLK\SDIO\NCS\VCC). Эти пины мы будем паять на плату переходник elite-c holder 2.1.
Распиновка следующая
| Плата сенсора | Плата переходник |
|---|---|
| GND | GND |
| VCC | 5V |
| MO | MISO |
| SCLK | SCLK |
| SDIO | MOSI |
| NCS | 101 pin on nice!nano(nrf52840) |

Далее переходим к редактированию прошивки. В моес случае это charybdis_3610.dtsi

Первый - значение по умолчанию, которое будет у Вас после форка проекта.
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
Второй блок кода, это то что нужно вставить. Сравните все значения, внесите себе те, которые отличаются от Ваших.
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
И добавить блок кода в низ файла
```
/ {
  trackball_listener {
    compatible = "zmk,input-listener";
    device = <&trackball>;

  };
};
```

Для более корректной работы шара, вставьте в конфигурацию правой или левой половинки, тот что лежит в директории /config/ следующие параметры 

```
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_PMW3610=y
CONFIG_ZMK_MOUSE=y
CONFIG_ZMK_EXT_POWER=y
CONFIG_PMW3610_SWAP_XY=y
CONFIG_PMW3610_INVERT_X=y
CONFIG_PMW3610_INVERT_Y=y
CONFIG_PMW3610_REST1_SAMPLE_TIME_MS=20
CONFIG_PMW3610_REST2_SAMPLE_TIME_MS=200
CONFIG_PMW3610_REST3_SAMPLE_TIME_MS=300
CONFIG_PMW3610_RUN_DOWNSHIFT_TIME_MS=500
CONFIG_PMW3610_REST1_DOWNSHIFT_TIME_MS=3000
CONFIG_PMW3610_REST2_DOWNSHIFT_TIME_MS=30000
CONFIG_PMW3610_SMART_ALGORITHM=y
CONFIG_ZMK_POINTING=y
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
      
