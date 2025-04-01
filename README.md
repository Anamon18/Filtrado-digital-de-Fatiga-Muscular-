# Señales electromiográficas EMG - Filtrado digital
## Descripción
En el presente laboratorio se realiza el filtrado digital para una señal electromiografica del músculo del bicep izquierdo con el objetivo de identificar la fatiga muscular apartir del analisis espectral. Para ello, se utiliza el sistema de adquisisión de datos **DAQ** con ayuda de lenguaje de porgramación **Python**. 

El trabajo se divide en:
+ Adquisisicion de la señal.
+ Filtrado de la señal.
+ Aventanamiento.
+ Analisis espectral.
## Tener en cuenta
1. El músculo que se toma es el bicep izquierdo.
2. La duración de la adquisición es de 60 segundo.
3. La frecuencia de la señal electromiográfica *(EMG)* de un músculo puede variar entre 20 y 100 Hz, asi que utilizamos **2000Hz** para la frecuencia de muestreo.
4. Se debe instalar las librerias:
   + Csv.
   + Numpy.
   + Pandas.
   + Matplotlib.
5. Se utiliza **Jupyter NoteBook** para dividir el código en partes y trabajar en ellas sin importar el orden: escribir, probar funciones, cargar un archivo en la memoria y procesar el contenido. Con lenguaje de **Python**
## 1. Adquisición de datos 
Se configura una tarea para el modulo DAQ para que reciba los datos en una frecuencia de muestreo de 2000 Hz 
```python
#  Parámetros de adquisición
puerto= "Dev3/ai0"  # Según el canal de tu DAQ
fmuestreo = 2000  # Frecuencia de muestreo en Hz
t_ventana = 2000  # Cantidad de muestras a mostrar en la gráfica
duracion = 60  # Duración de la adquisición en segundos

#  Estructura de datos para almacenar la señal en tiempo real
emg_buffer = deque([0] * t_ventana, maxlen=t_ventana)  # Guarda los últimos datos capturados
tiempo_buffer = deque(np.linspace(0, (t_ventana - 1) / fmuestreo,t_ventana), maxlen=t_ventana)# Eje X desde 0

#  Guardar los datos
filename = "emg_data.csv"
with open(filename, mode='w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(["Tiempo (s)", "Voltaje (V)"])  # Escribir encabezado

    #  Tarea de adquisición
    with nidaqmx.Task() as task:
        task.ai_channels.add_ai_voltage_chan(puerto)
        task.timing.cfg_samp_clk_timing(fmuestreo)

        print("Adquiriendo datos en tiempo real... Presiona 'Ctrl + C' para detener.")

        #  Inicializar la figura
        plt.ion()  # Modo interactivo
        fig, ax = plt.subplots(figsize=(10, 4))
        line, = ax.plot(tiempo_buffer, emg_buffer, label="Señal EMG")
        ax.set_xlabel("Tiempo (s)")
        ax.set_ylabel("Voltaje (V)")
        ax.set_title("Señal EMG en Tiempo Real")
        ax.legend()
        ax.grid(True)
        ax.set_ylim([-1, 5])  # Ajuste de voltaje de -1V a 1V

        #  Bucle de adquisición en tiempo real
        try:
            total_samples = int(duracion * fmuestreo)
            for i in range(0, total_samples, t_ventana):
                # Leer múltiples muestras a la vez
                emg_data = task.read(number_of_samples_per_channel=t_ventana)

                if isinstance(emg_data, list):  # Confirmar que es una lista
                    emg_buffer.extend(emg_data)

                    # Guardar los datos en el archivo
                    for j in range(len(emg_data)):
                        tiempo = (i + j) / fmuestreo
                        writer.writerow([tiempo, emg_data[j]])
```
>Esta parte del código permite almacenar los datos de 60s y guardarlos en un archivo **.csv** para luego poder realizar un DataFrame y visualizar el voltaje y el tiempo para graficar y poder aplicar el filtro digital.
### 1.2  Importar datos.
```python
import pandas as pd
import csv

# Cargar datos y mostrar en un DataFrame
filename = "emg_data.csv"
df = pd.read_csv(filename)
print(df.head(60))

plt.figure(figsize=(17, 10))  # Tamaño del gráfico
plt.plot(df["Tiempo (s)"], df["Voltaje (V)"], label="Fatiga muscular", color="pink")
plt.title("Fatiga Muscular", fontsize=16)
plt.xlabel("Tiempo (s)", fontsize=14)
plt.ylabel("Amplitud (mV)", fontsize=14)
plt.grid(True, linestyle="--", alpha=0.7)
plt.legend(fontsize=12)
plt.savefig("Fatiga.png", dpi=300, bbox_inches='tight')
plt.show()
```
![image](https://github.com/user-attachments/assets/09dac390-3658-4668-92ff-8c5effd16487)
<br><em>Figura 1: Fatiga muscular del bicep sin filtrar. mV vs t(s) .</em></p>
