import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3

# Conecta ou cria banco
conn = sqlite3.connect("clientes.db")
cursor = conn.cursor()
cursor.execute('''
    CREATE TABLE IF NOT EXISTS clientes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nome TEXT NOT NULL,
        cpf TEXT NOT NULL,
        email TEXT NOT NULL UNIQUE,
        telefone TEXT NOT NULL,
        senha TEXT NOT NULL
    )
''')
conn.commit()

# Funções principais
def criar_conta():
    def salvar_dados():
        nome = entry_nome.get()
        cpf = entry_cpf.get()
        email = entry_email.get()
        telefone = entry_telefone.get()
        senha = entry_senha.get()

        if nome and cpf and email and telefone and senha:
            try:
                cursor.execute("INSERT INTO clientes (nome, cpf, email, telefone, senha) VALUES (?, ?, ?, ?, ?)",
                               (nome, cpf, email, telefone, senha))
                conn.commit()
                messagebox.showinfo("Sucesso", "Conta criada com sucesso!")
                janela_cadastro.destroy()
            except sqlite3.IntegrityError:
                messagebox.showerror("Erro", "E-mail já cadastrado.")
        else:
            messagebox.showwarning("Aviso", "Preencha todos os campos.")

    janela_cadastro = tk.Toplevel(janela)
    janela_cadastro.title("Criar Conta")

    tk.Label(janela_cadastro, text="Nome").grid(row=0, column=0)
    tk.Label(janela_cadastro, text="CPF").grid(row=1, column=0)
    tk.Label(janela_cadastro, text="E-mail").grid(row=2, column=0)
    tk.Label(janela_cadastro, text="Telefone").grid(row=3, column=0)
    tk.Label(janela_cadastro, text="Senha").grid(row=4, column=0)

    entry_nome = tk.Entry(janela_cadastro)
    entry_cpf = tk.Entry(janela_cadastro)
    entry_email = tk.Entry(janela_cadastro)
    entry_telefone = tk.Entry(janela_cadastro)
    entry_senha = tk.Entry(janela_cadastro, show="*")

    entry_nome.grid(row=0, column=1)
    entry_cpf.grid(row=1, column=1)
    entry_email.grid(row=2, column=1)
    entry_telefone.grid(row=3, column=1)
    entry_senha.grid(row=4, column=1)

    tk.Button(janela_cadastro, text="Salvar", command=salvar_dados).grid(row=5, column=0, columnspan=2)

def login():
    def verificar_login():
        email = entry_email.get()
        senha = entry_senha.get()
        cursor.execute("SELECT * FROM clientes WHERE email=? AND senha=?", (email, senha))
        usuario = cursor.fetchone()
        if usuario:
            janela_login.destroy()
            mostrar_painel_administrador()
        else:
            messagebox.showerror("Erro", "Login inválido.")

    janela_login = tk.Toplevel(janela)
    janela_login.title("Login")

    tk.Label(janela_login, text="E-mail").grid(row=0, column=0)
    tk.Label(janela_login, text="Senha").grid(row=1, column=0)

    entry_email = tk.Entry(janela_login)
    entry_senha = tk.Entry(janela_login, show="*")

    entry_email.grid(row=0, column=1)
    entry_senha.grid(row=1, column=1)

    tk.Button(janela_login, text="Entrar", command=verificar_login).grid(row=2, column=0, columnspan=2)

def mostrar_painel_administrador():
    painel = tk.Toplevel(janela)
    painel.title("Painel de Administração")

    tree = ttk.Treeview(painel, columns=("ID", "Nome", "CPF", "Email", "Telefone"), show="headings")
    tree.heading("ID", text="ID")
    tree.heading("Nome", text="Nome")
    tree.heading("CPF", text="CPF")
    tree.heading("Email", text="E-mail")
    tree.heading("Telefone", text="Telefone")
    tree.pack()

    def carregar_usuarios():
        for row in tree.get_children():
            tree.delete(row)
        cursor.execute("SELECT id, nome, cpf, email, telefone FROM clientes")
        for usuario in cursor.fetchall():
            tree.insert("", "end", values=usuario)

    def deletar_usuario():
        item = tree.selection()
        if item:
            usuario_id = tree.item(item, "values")[0]
            cursor.execute("DELETE FROM clientes WHERE id=?", (usuario_id,))
            conn.commit()
            carregar_usuarios()
            messagebox.showinfo("Sucesso", "Usuário deletado com sucesso.")
        else:
            messagebox.showwarning("Aviso", "Selecione um usuário para deletar.")

    carregar_usuarios()
    tk.Button(painel, text="Deletar Usuário Selecionado", command=deletar_usuario).pack(pady=10)

# Janela principal
janela = tk.Tk()
janela.title("Sistema de Cadastro de Clientes")
janela.geometry("300x200")

tk.Button(janela, text="Criar Conta", width=20, command=criar_conta).pack(pady=10)
tk.Button(janela, text="Entrar", width=20, command=login).pack(pady=10)

janela.mainloop()

