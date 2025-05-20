import tkinter as tk
from tkinter import messagebox
import re
class NodoTurno:
    def __init__(self, descripcion):
        self.descripcion = descripcion  
        self.izquierda = None  
        self.derecha = None    

def es_movimiento_valido(mov):
    patron = r'^(O-O(-O)?|[KQRBN]?[a-h]?[1-8]?x?[a-h][1-8](=[QRBN])?[+#]?)$'
    return re.match(patron, mov) is not None


def construir_arbol_binario(jugadas):
    raiz = NodoTurno("Partida")
    actual_turno = raiz

    for jugada in jugadas:
        match = re.match(r'^(\d+)\.\s*([^\s]+)(?:\s+([^\s]+))?$', jugada.strip())
        if not match:
            raise ValueError(f"Formato inválido: {jugada}")

        numero, mov_blancas, mov_negras = match.groups()

        if not es_movimiento_valido(mov_blancas):
            raise ValueError(f"Movimiento inválido de blancas: {mov_blancas}")

        if mov_negras and not es_movimiento_valido(mov_negras):
            raise ValueError(f"Movimiento inválido de negras: {mov_negras}")

        nodo_turno = NodoTurno(f"Turno {numero}")
        nodo_turno.izquierda = NodoTurno(mov_blancas)
        if mov_negras:
            nodo_turno.derecha = NodoTurno(mov_negras)

        if not actual_turno.izquierda:
            actual_turno.izquierda = nodo_turno
        else:
            temp = actual_turno.izquierda
            while temp.izquierda:
                temp = temp.izquierda
            temp.izquierda = nodo_turno

    return raiz


def imprimir_arbol_binario(nodo, prefijo=""):
    if not nodo:
        return ""
    resultado = f"{prefijo}{nodo.descripcion}\n"
    resultado += imprimir_arbol_binario(nodo.izquierda, prefijo + "  ")
    resultado += imprimir_arbol_binario(nodo.derecha, prefijo + "  ")
    return resultado


class App:
    def __init__(self, root):
        self.root = root
        self.root.title("Validador SAN y Árbol Ajedrez")

        self.texto = tk.Text(root, height=20, width=60)
        self.texto.pack(padx=10, pady=10)

        self.boton = tk.Button(root, text="Validar y construir árbol", command=self.procesar_entrada)
        self.boton.pack(pady=5)

    def procesar_entrada(self):
        entrada = self.texto.get("1.0", tk.END).strip().split('\n')
        try:
            arbol = construir_arbol_binario(entrada)
            resultado = imprimir_arbol_binario(arbol)
            messagebox.showinfo("Árbol generado", resultado)
        except Exception as e:
            messagebox.showerror("Error", str(e))

if __name__ == "__main__":
    root = tk.Tk()
    app = App(root)
    root.mainloop()
