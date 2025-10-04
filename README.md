Neste projeto didático com **sensor AHT10** e **display OLED SSD1306**, integrados via **I²C duplo**, mostrando do uso do **Raspberry Pi Pico w** com o **Pico SDK em C**.


---

## 🧠 **Explicação do Projeto**

### 📋 **Objetivo**

Ler os valores de **temperatura e umidade** do sensor **AHT10**, exibir os dados em um **display OLED 128x64 (SSD1306)** e emitir **alertas visuais** no display quando:

* A **umidade** for **superior a 70%**
* A **temperatura** for **inferior a 20 °C**

---

## ⚙️ **Componentes e Conexões**

| Componente          | Função                     | Interface  | Pinos do Pico              |
| ------------------- | -------------------------- | ---------- | -------------------------- |
| AHT10               | Mede temperatura e umidade | I²C (i2c0) | SDA = GPIO0, SCL = GPIO1   |
| SSD1306 OLED 128x64 | Mostra as informações      | I²C (i2c1) | SDA = GPIO14, SCL = GPIO15 |

---

## 🧩 **Principais Blocos do Código**

### 🧱 **1. Inicialização dos I²C**

```c
i2c_init(I2C_PORT0, 100 * 1000);  // Sensor AHT10 a 100kHz
i2c_init(I2C_PORT1, 400000);      // OLED a 400kHz
```

São usadas **duas interfaces I²C independentes** para evitar conflito de endereços e melhorar o desempenho do display.

---

### 💾 **2. Inicialização do Display OLED**

```c
ssd1306_init(I2C_PORT1);
ssd1306_clear();
ssd1306_draw_string(32, 0, "Sensor AHT10");
ssd1306_show();
```

Exibe a tela inicial com o título “Sensor AHT10”.

---

### 🌡️ **3. Inicialização do Sensor AHT10**

```c
AHT10_Handle aht10 = {
    .iface = {
        .i2c_write = i2c_write,
        .i2c_read = i2c_read,
        .delay_ms = delay_ms
    }
};
AHT10_Init(&aht10);
```

Cria uma estrutura de controle para o sensor e associa as funções de comunicação I²C implementadas no final do código.

Se a inicialização falhar, o programa mostra no display:

```
Sensor AHT10
Falha no AHT10
```

e permanece travado no loop de erro.

---

### 🔁 **4. Loop Principal**

A cada ciclo o programa:

1. **Lê temperatura e umidade**

   ```c
   AHT10_ReadTemperatureHumidity(&aht10, &temp, &hum);
   ```

2. **Verifica condições de alerta**

   * **Umidade acima de 70%**
     Mostra aviso piscando:

     ```
     Umidade 72.5 %
     Acima de 70 %
     ATENÇÃO
     ```
   * **Temperatura abaixo de 20°C**
     Mostra aviso piscando:

     ```
     Temperatura 18.3 C
     Abaixo de 20 C
     ATENÇÃO
     ```

   O alerta pisca na tela até que a condição volte ao normal.

3. **Mostra leitura normal**

   ```
   Sensor AHT10
   Temperatura 23.5 C
   Umidade 58.4 %
   ```

   Atualizando a cada 1 segundo (`sleep_ms(1000)`).

---

## 📟 **Funções Auxiliares**

| Função        | Descrição                                     |
| ------------- | --------------------------------------------- |
| `i2c_write()` | Envia dados ao sensor AHT10                   |
| `i2c_read()`  | Lê dados do sensor AHT10                      |
| `delay_ms()`  | Aguarda o tempo especificado em milissegundos |

Essas funções são repassadas à biblioteca do sensor, permitindo seu funcionamento genérico com o hardware do Pico.

---

## 💡 **Resultados Esperados**

| Situação          | Leitura         | Exibição no OLED                      | Ação                         |
| ----------------- | --------------- | ------------------------------------- | ---------------------------- |
| Normal            | 22.3 °C / 60.5% | “Temperatura 22.3 C / Umidade 60.5 %” | Exibição contínua            |
| Umidade alta      | 25.0 °C / 75.2% | “Acima de 70% — ATENÇÃO” piscando     | Alerta visual até normalizar |
| Temperatura baixa | 18.5 °C / 65.0% | “Abaixo de 20°C — ATENÇÃO” piscando   | Alerta visual até normalizar |

No **terminal serial**, as leituras também aparecem:

```
Temperatura: 22.45 °C | Umidade: 58.72 %
Temperatura: 18.93 °C | Umidade: 61.40 %
```

---

## ✅ **Resumo Técnico**

| Item                | Detalhe                           |
| ------------------- | --------------------------------- |
| Placa               | BitDogLAb/Raspberry Pi Pico       |
| Sensor              | AHT10 (I²C)                       |
| Display             |BitDogLab/ OLED SSD1306 (I²C)      |
| Linguagem           | C (Pico SDK)                      |
| Taxa de atualização | 1 segundo                         |
| Funções extras      | Alertas visuais automáticos       |
| Critérios de alerta | Umidade > 70%, Temperatura < 20°C |

---


