# Los-filosofos

Este es el link del repositorio: [GitHub](https://github.com/alexlomu/Los-filosofos)

El código proporcionado es el siguiente:

    import time
    import random
    import threading
    import tkinter as tk


    N = 5
    TIEMPO_TOTAL = 3


    class filosofo(threading.Thread):
        semaforo = threading.Lock()
        estado = []
        tenedores = []
        count = 0

        def __init__(self, gui):
            super().__init__()
            self.gui = gui
            self.id = filosofo.count
            filosofo.count += 1
            filosofo.estado.append('PENSANDO')
            filosofo.tenedores.append(threading.Semaphore(0))
            self.gui.actualizar_estado(self.id, 'PENSANDO')
            self.gui.actualizar_comidas(self.id, 0)
            self.gui.agregar_log("FILOSOFO {} - PENSANDO".format(self.id))

        def __del__(self):
            self.gui.agregar_log("FILOSOFO {} - Se para de la mesa".format(self.id))

        def pensar(self):
            time.sleep(random.randint(0, 5))

        def derecha(self, i):
            return (i - 1) % N

        def izquierda(self, i):
            return (i + 1) % N

        def verificar(self, i):
            if filosofo.estado[i] == 'HAMBRIENTO' and filosofo.estado[self.izquierda(i)] != 'COMIENDO' and filosofo.estado[
                self.derecha(i)] != 'COMIENDO':
                filosofo.estado[i] = 'COMIENDO'
                filosofo.tenedores[i].release()

        def tomar(self):
            filosofo.semaforo.acquire()
            filosofo.estado[self.id] = 'HAMBRIENTO'
            self.verificar(self.id)
            filosofo.semaforo.release()
            filosofo.tenedores[self.id].acquire()
            self.gui.actualizar_estado(self.id, 'COMIENDO')
            self.gui.agregar_log("FILOSOFO {} - Entra a comer".format(self.id))

        def soltar(self):
            filosofo.semaforo.acquire()
            filosofo.estado[self.id] = 'PENSANDO'
            self.verificar(self.izquierda(self.id))
            self.verificar(self.derecha(self.id))
            filosofo.semaforo.release()
            self.gui.actualizar_estado(self.id, 'PENSANDO')
            self.gui.agregar_log("FILOSOFO {} - Sale de comer".format(self.id))

        def comer(self):
            self.gui.aumentar_comidas(self.id)
            self.gui.actualizar_comidas(self.id, self.gui.obtener_comidas(self.id) + 1)
            self.gui.agregar_log("FILOSOFO {} - Comiendo".format(self.id))
            time.sleep(2)
            self.gui.agregar_log("FILOSOFO {} - Terminó de comer".format(self.id))

        def run(self):
            for i in range(TIEMPO_TOTAL):
                self.pensar()
                self.tomar()
                self.comer()
                self.soltar()


    class CenaFilosofosGUI(tk.Tk):
        def __init__(self):
            super().__init__()
            self.title("Cena de los Filósofos")
            self.geometry("800x600")

            self.filosofos_labels = []
            self.tenedores_labels = []
            self.estado_labels = []
            self.comidas_labels = []

            self.log_text = tk.Text(self, height=10, width=60)
            self.log_text.pack()

            self.controles_frame = tk.Frame(self)
            self.controles_frame.pack(side=tk.BOTTOM)

            self.iniciar_btn = tk.Button(self.controles_frame, text="Iniciar", command=self.iniciar)
            self.iniciar_btn.pack(side=tk.LEFT)

            self.pausar_btn = tk.Button(self.controles_frame, text="Pausar", command=self.pausar)
            self.pausar_btn.pack(side=tk.LEFT)

            self.reset_btn = tk.Button(self.controles_frame, text="Reset", command=self.reset)
            self.reset_btn.pack(side=tk.LEFT)

            self.salir_btn = tk.Button(self.controles_frame, text="Salir", command=self.salir)
            self.salir_btn.pack(side=tk.LEFT)

            self.creditos_btn = tk.Button(self.controles_frame, text="Créditos", command=self.creditos)
            self.creditos_btn.pack(side=tk.LEFT)

            self.inicializar_interfaz()

        def inicializar_interfaz(self):
            for i in range(N):
                filosofo_label = tk.Label(self, text="Filósofo {}".format(i))
                filosofo_label.place(x=200 + i * 100, y=100)
                self.filosofos_labels.append(filosofo_label)

                tenedor_label = tk.Label(self, text="Tenedor")
                tenedor_label.place(x=180 + i * 100, y=150)
                self.tenedores_labels.append(tenedor_label)

                estado_label = tk.Label(self, text="PENSANDO", fg="blue")
                estado_label.place(x=200 + i * 100, y=200)
                self.estado_labels.append(estado_label)

                comidas_label = tk.Label(self, text="Comidas: 0")
                comidas_label.place(x=200 + i * 100, y=250)
                self.comidas_labels.append(comidas_label)

        def actualizar_estado(self, filosofo_id, estado):
            color = "blue"  # Color para el estado "PENSANDO"
            if estado == "COMIENDO":
                color = "green"
            elif estado == "HAMBRIENTO":
                color = "red"

            self.estado_labels[filosofo_id].configure(text=estado, fg=color)

        def agregar_log(self, mensaje):
            self.log_text.insert(tk.END, mensaje + "\n")
            self.log_text.see(tk.END)

        def actualizar_comidas(self, filosofo_id, comidas):
            self.comidas_labels[filosofo_id].configure(text="Comidas: {}".format(comidas))

        def obtener_comidas(self, filosofo_id):
            comidas_text = self.comidas_labels[filosofo_id].cget("text")
            comidas = int(comidas_text.split(":")[1].strip())
            return comidas

        def aumentar_comidas(self, filosofo_id):
            comidas = self.obtener_comidas(filosofo_id)
            comidas += 1
            self.actualizar_comidas(filosofo_id, comidas)

        def iniciar(self):
            self.filosofos = []
            for i in range(N):
                filosofo_obj = filosofo(self)
                self.filosofos.append(filosofo_obj)
                filosofo_obj.start()

            self.iniciar_btn.configure(state=tk.DISABLED)
            self.pausar_btn.configure(state=tk.NORMAL)

        def pausar(self):
            for filosofo_obj in self.filosofos:
                filosofo_obj.pause()

            self.pausar_btn.configure(text="Reanudar", command=self.reanudar)

        def reanudar(self):
            for filosofo_obj in self.filosofos:
                filosofo_obj.resume()

            self.pausar_btn.configure(text="Pausar", command=self.pausar)

        def reset(self):
            self.log_text.delete(1.0, tk.END)
            for i in range(N):
                self.actualizar_estado(i, "PENSANDO")
                self.actualizar_comidas(i, 0)

        def salir(self):
            self.destroy()

        def creditos(self):
            tk.messagebox.showinfo("Créditos", "Desarrollado por [Tu nombre]")


    if __name__ == "__main__":
        ventana = CenaFilosofosGUI()
        ventana.mainloop()

La interfaz funcionando se ve así:

![image](https://github.com/alexlomu/Los-filosofos/assets/91721507/1ed20568-70e3-41ea-b3b7-0e4982e3ffbd)

