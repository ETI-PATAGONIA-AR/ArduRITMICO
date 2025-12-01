# ArduRítmico V3.3
Analizador de audio y visualizador en tiempo real basado en Arduino. Convierte la música en señales luminosas y en una visualización gráfica en un display OLED.

## 🎧 ¿Qué hace este proyecto?
ArduRítmico “escucha” la música del ambiente mediante un sensor de sonido y:

- Analiza sus frecuencias en tiempo real
- Enciende luces según qué rangos predominan
- Muestra un analizador de espectro en una OLED

[![Ver demo en YouTube](https://img.youtube.com/vi/TU4yZVd3JM4/0.jpg)](https://www.youtube.com/watch?v=TU4yZVd3JM4)


## ⚙️ ¿Cómo funciona internamente?
El ArduRítmico V3.3 básicamente es un analizador de audio en tiempo real (entre nos, casi en tiempo rela)... Lo arme tanto con un PIC, como con Arduino y utiliza FFT 
para segmentar el espectro en tres bandas de frecuencia: bajos, medios y agudos. Cada banda controla independientemente salidas digitales para iluminación y LEDs RGB, 
generando un sistema de respuesta lumínica sincronizada con la música.... En este ejemplo puse led, pero tranquilamente pueden ir control de motores, servos, etc...
El sistema adquiere la señal del sensor de sonido (un simple Shield que sale menos de  $4.500 aproximadamente) mediante el conversor ADC del microcontrolador. Se ajusta 
el prescaler del ADC para obtener una frecuencia de muestreo cercana a los 38 kHz, lo cual permite analizar señales de audio de hasta ~19 kHz cumpliendo el teorema de 
Nyquist (y hasta por ahí nomas por que la pantalla TFT o el ADC no corre en segundo plano, asi que algo se pierde en el camino...es tan rápido que no es perceptible a 
los ojos). Además, se utiliza la referencia interna de 1.1 V para aumentar la resolución en señales de baja amplitud... Tomo 128 muestras por ciclo de análisis. Antes de 
procesarlas se aplica una ventana de Hann para reducir el efecto de las discontinuidades en los extremos del bloque de datos; Luego se ejecuta la transformación rápida de 
Fourier usando la librería fix_fft, con los resultados almacenados en arrays separados para la parte real e imaginaria.
Posteriormente se calcula el módulo de cada componente espectral para obtener la magnitud por banda. Se suman los valores correspondientes a los rangos que representan 
graves (2 a 7), medios (7 a 35) y agudos (35 a 64). Los valores promediados se comparan con límites adaptativos: después de un número determinado de pasadas, dichos límites 
se recalculan en función de cuántas veces se superó cada banda, logrando un autoajuste frente a variaciones de volumen en la entrada... Si una banda supera el límite, se 
activa la salida digital correspondiente y también un LED RGB asignado a cada frecuencia (rojo para bajos, verde para medios y azul para agudos). Esto permite una clara 
separación visual entre componentes sonoras.
También incorpore una interfaz gráfica con display OLED SSD1306, que luego de haberlo terminado me arrepenti. En cada ciclo se realiza una segunda FFT para generar un 
analizador de espectro simplificado, donde se dibujan barras verticales que representan la intensidad de las frecuencias en tiempo real, junto con una barra horizontal que 
indica el nivel general del sonido. Toda la visualización se refresca continuamente para una respuesta fluida.

### 1️⃣ Captura de audio
```cpp
for (int i = 0; i < MUESTRAS; i++) {
    data[i] = analogRead(A0)/4 - 128;
    im[i] = 0;
}
```

### 2️⃣ Ventaneado (Hann)
Reduce efectos no deseados en los extremos del muestreo.
```cpp
aplicarVentanaHann(data);
```

### 3️⃣ FFT (Transformada Rápida de Fourier)
Convierte tiempo → frecuencia para identificar energía espectral.
```cpp
fix_fft(data, im, LOGM, 0);
```

Se obtiene magnitud de cada banda:
```cpp
magnitud[i] = sqrt(data[i] * data[i] + im[i] * im[i]);
```

### 4️⃣ Clasificación por bandas

Las frecuencias se agrupan así:

Rango FFT -	Banda -	Sonido
2 a 7	    -Bajos	 -Golpes, graves
8 a 35	  -Medios	 -Voz, instrumentos principales
36 a 64	  -Agudos	 -Platillos, brillo del sonido

```cpp
bajos  = energia(2, 7);
medios = energia(8, 35);
agudos = energia(36, 64);
```

### 5️⃣ Control de salidas luminosas
Se comparan energías con límites adaptativos (evita saturación).

```cpp
digitalWrite(PIN_BAJOS,  bajos  > limBajos);
digitalWrite(PIN_MEDIOS, medios > limMedios);
digitalWrite(PIN_AGUDOS, agudos > limAgudos);
```
Si una banda supera su umbral → ¡esa luz se enciende!

**Librerías utilizadas:**

_Adafruit GFX_
_Adafruit SSD1306_
_fix_fft (agregar manualmente)_

<img width="4531" height="3031" alt="ArduRITMICO_CIRCUIT" src="https://github.com/user-attachments/assets/dbdf2575-487a-45d1-a932-59734c2b1eaf" />

<img width="819" height="538" alt="ArduRITMICO_3D_1" src="https://github.com/user-attachments/assets/d6514ca5-d038-451d-96d3-e97c6f0d17f3" />

<img width="555" height="548" alt="ArduRITMICO_PCB2" src="https://github.com/user-attachments/assets/77f0b1b3-6b82-498d-9136-d35b1c25190c" />

<img width="530" height="573" alt="ArduRITMICO_PCB1" src="https://github.com/user-attachments/assets/d412f1c7-70fc-4717-928e-780e3e21d42b" />

🧑‍💻 ****El Código completo?
El código completo no se incluye en el repositorio para proteger el desarrollo del proyecto.
📌 Disponibilidad solo con fines educativos y bajo solicitud del autor...
Si querés recibirlo:
📧 prof.martintorres@educ.ar






