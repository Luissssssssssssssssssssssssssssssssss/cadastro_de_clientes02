# cadastro_de_clientes02
import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3
import os

# ---------- BANCO DE DADOS ----------
def criar_banco():
    if not os.path.exists("clientes.db"):
        conn = sqlite3.connect("clientes.db")
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS clientes (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nome TEXT,
                cpf TEXT,
                email TEXT UNIQUE,
                telefone TEXT,
                senha TEXT
            )
        """)
        conn.commit()
        conn.close()

# ---------- FUNÇÕES DE INTERFACE ----------
def tela_inicial():
    limpar_tela()
    tk.Label(janela, text="Bem-vindo!", font=("Arial", 16)).pack(pady=20)
    tk.Button(janela, text="Criar Conta", width=30, command=tela_cadastro).pack(pady=10)
    tk.Button(janela, text="Entrar", width=30, command=tela_login).pack(pady=10)

def tela_cadastro():
    limpar_tela()
    tk.Label(janela, text="Cadastro de Cliente", font=("Arial", 14)).pack(pady=10)

    entradas.clear()
    campos = ["Nome", "CPF", "Email", "Telefone", "Senha"]
    for campo in campos:
        tk.Label(janela, text=campo).pack()
        entrada = tk.Entry(janela, show="*" if campo == "Senha" else None, width=40)
        entrada.pack()
        entradas[campo] = entrada

    tk.Button(janela, text="Cadastrar", command=cadastrar_cliente, bg="green", fg="white").pack(pady=15)
    tk.Button(janela, text="Voltar", command=tela_inicial).pack()

def tela_login():
    limpar_tela()
    tk.Label(janela, text="Login", font=("Arial", 14)).pack(pady=10)

    entradas.clear()
    for campo in ["Email", "Senha"]:
        tk.Label(janela, text=campo).pack()
        entrada = tk.Entry(janela, show="*" if campo == "Senha" else None, width=40)
        entrada.pack()
        entradas[campo] = entrada

    tk.Button(janela, text="Entrar", command=logar_cliente, bg="blue", fg="white").pack(pady=15)
    tk.Button(janela, text="Voltar", command=tela_inicial).pack()

def tela_painel():
    limpar_tela()
    tk.Label(janela, text="Clientes Cadastrados", font=("Arial", 14)).pack(pady=10)

    tree = ttk.Treeview(janela, columns=("Nome", "CPF", "Email", "Telefone"), show="headings")
    for col in ("Nome", "CPF", "Email", "Telefone"):
        tree.heading(col, text=col)
        tree.column(col, width=120)
    tree.pack()

    conn = sqlite3.connect("clientes.db")
    cursor = conn.cursor()
    cursor.execute("SELECT nome, cpf, email, telefone FROM clientes")
    for linha in cursor.fetchall():
        tree.insert("", tk.END, values=linha)
    conn.close()

    tk.Button(janela, text="Voltar", command=tela_inicial).pack(pady=10)

# ---------- AÇÕES ----------
def cadastrar_cliente():
    dados = {campo: entrada.get() for campo, entrada in entradas.items()}
    if not all(dados.values()):
        messagebox.showwarning("Erro", "Preencha todos os campos.")
        return

    conn = sqlite3.connect("clientes.db")
    cursor = conn.cursor()
    try:
        cursor.execute("INSERT INTO clientes (nome, cpf, email, telefone, senha) VALUES (?, ?, ?, ?, ?)",
                       (dados["Nome"], dados["CPF"], dados["Email"], dados["Telefone"], dados["Senha"]))
        conn.commit()
        messagebox.showinfo("Sucesso", "Cadastro salvo com sucesso!")
        tela_inicial()
    except sqlite3.IntegrityError:
        messagebox.showerror("Erro", "E-mail já cadastrado.")
    finally:
        conn.close()

def logar_cliente():
    email = entradas["Email"].get()
    senha = entradas["Senha"].get()

    conn = sqlite3.connect("clientes.db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM clientes WHERE email = ? AND senha = ?", (email, senha))
    resultado = cursor.fetchone()
    conn.close()

    if resultado:
        tela_painel()
    else:
        messagebox.showerror("Erro", "Email ou senha incorretos.")

def limpar_tela():
    for widget in janela.winfo_children():
        widget.destroy()

# ---------- INTERFACE PRINCIPAL ----------
entradas = {}
janela = tk.Tk()
janela.title("Cadastro de Clientes")
janela.geometry("500x500")

criar_banco()
tela_inicial()
janela.mainloop()
