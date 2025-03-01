# Laboratorio 3 Señales

## SEPARACION DE AUDIOS METODO ICA Y BEAMFORMED
En primer lugar debemos comprender que los audios grabados no son de muy buena calidad debido a que se grabaron a una distancia de 6.096 m con una incertidumbre de 0.001 m debido a que fue medido en cm y ya en su conversion a metros mantiene en cierta manera la misma incertidumbre

Ahora bien para ello debemos tener en cuenta que paea ka separacion de audios, comprendemos que existe un ruido en el ambiente o en el medio, bien sea el aire, gritos, que otra persona grito, etc esto hace que debamos averiguar cual es la relacion entre el ruido del ambiente y el audio, esto se calculara con el SNR dicho bien este SNR se dara mediante la siguiente funcion calcular el snr mediante el audio de entrada.

Esta funcion se llamara "calculate_snr" en donde en dicha funcion entra un audio bien sea el de Jose o el de Jhonathan, para ello primero deberemos calcular el poder de audio esto se dara medianto la funcion np.mean del audio original al cuadrado, ya luego la 

        def calculate_snr(audio):
            audio_power = np.mean(audio ** 2)
            noise_power = np.var(audio - np.mean(audio)) 
            snr = 10 * np.log10(audio_power / noise_power)
            return snr

        snr_jhonathan = calculate_snr(Jhonathan)
        snr_jose = calculate_snr(Jose)

        print(f"SNR de Jhonathan: {snr_jhonathan:.2f} dB")
        print(f"SNR de José: {snr_jose:.2f} dB")

        t = np.linspace(0, 1, fs, endpoint=False)  # Vector de tiempo
        N = len(t)

        frequencies = np.fft.fftfreq(N, 1/fs)
        spectrum_Jhonathan = np.fft.fft(Jhonathan) / N
        magnitude_Jhonathan = 2 * np.abs(spectrum_Jhonathan[:N//2])
        spectrum_Jose = np.fft.fft(Jose) / N
        magnitude_Jose = 2 * np.abs(spectrum_Jose[:N//2])

        plt.figure(figsize=(12,4))
        plt.subplot(2, 1, 1)
        plt.plot(frequencies[:N//2], magnitude_Jhonathan, 'blue')
        plt.xlabel('Frecuencia (Hz)')
        plt.ylabel('Decibeles (db)')
        plt.title('Espectro de la señal normalizado')
        plt.grid()

        plt.subplot(2, 1, 2)
        plt.plot(frequencies[:N//2], magnitude_Jose, 'red')
        plt.xlabel('Frecuencia (Hz)')
        plt.ylabel('Decibeles (db)')
        plt.title('Espectro de la señal normalizado')
        plt.grid()

        plt.tight_layout()
        plt.show()

        # Separación con ICA
        minutos = min(len(audio_Jhonathan), len(audio_Jose))
        nuevo_audio_Jhonathan = audio_Jhonathan[:minutos]
        nuevo_audio_Jose = audio_Jose[:minutos]
        audios_mixtos = np.column_stack((nuevo_audio_Jhonathan, nuevo_audio_Jose))
        ica = FastICA(n_components=2)
        audios_separados = ica.fit_transform(audios_mixtos)
        audios_separados = np.int16(audios_separados / np.max(np.abs(audios_separados)) * 32767)

        wav.write(output_dir + "Audio_ICA_Jhonathan.wav", fs, audios_separados[:, 0])
        wav.write(output_dir + "Audio_ICA_Jose.wav", fs, audios_separados[:, 1])

        print("Se guardaron las señales filtradas con ICA")

        # Implementación de Beamforming (Delay-and-Sum)
        def delay_and_sum(audio_signals, delays):
            audio_signals = [np.array(audio, dtype=np.float32).flatten() for audio in audio_signals]
            min_length = min(map(len, audio_signals))
            audio_signals = [audio[:min_length] for audio in audio_signals]
            max_delay = max(delays)
            delayed_signals = [
                np.pad(audio, (delay, max_delay - delay), mode='constant')[:min_length]
                for audio, delay in zip(audio_signals, delays)
            ]
            return np.mean(delayed_signals, axis=0)

        # Simulación de retardos para Beamforming (en muestras)
        delays = [0, 5]  # Ajusta según la posición de los micrófonos
        beamformed_Jhonathan = delay_and_sum([audio_Jhonathan, audio_Jose], delays)
        beamformed_Jose = delay_and_sum([audio_Jose, audio_Jhonathan], delays)

        beamformed_Jhonathan = np.int16(beamformed_Jhonathan / np.max(np.abs(beamformed_Jhonathan)) * 32767)
        beamformed_Jose = np.int16(beamformed_Jose / np.max(np.abs(beamformed_Jose)) * 32767)

        # Guardar los audios procesados con Beamforming
        wav.write(output_dir + "Audio_Beamforming_Jhonathan.wav", fs, beamformed_Jhonathan)
        wav.write(output_dir + "Audio_Beamforming_Jose.wav", fs, beamformed_Jose)

        print("Se guardaron las señales filtradas con Beamforming")
