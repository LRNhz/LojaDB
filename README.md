# LojaDB
Não sei como anexa direito o arquivo do SQL, mas realizei a tarefa e deu tudo certo, segue o comando que utilizei para criar o Banco de dados: 

USE LojaDB;

-- Criar sequence para geração de identificadores de pessoa
GO
CREATE SEQUENCE Pessoa_Sequence
START WITH 1
INCREMENT BY 1;
GO

-- Criar procedure para obter próximo ID
GO
CREATE PROCEDURE dbo.GetNextPessoaId
    @id INT OUTPUT
AS
BEGIN
    SET @id = NEXT VALUE FOR Pessoa_Sequence;
END;
GO

-- Criar tabelas
GO
CREATE TABLE Usuarios (
  id INT PRIMARY KEY IDENTITY(1,1),
  username VARCHAR(50) NOT NULL,
  senha VARCHAR(255) NOT NULL,
  nome VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL
);

GO
CREATE TABLE Pessoas (
  id INT PRIMARY KEY,
  tipo VARCHAR(10) NOT NULL CHECK (tipo IN ('física', 'jurídica')),
  nome VARCHAR(100) NOT NULL,
  identificação VARCHAR(20) NOT NULL,
  endereço VARCHAR(200) NOT NULL,
  telefone VARCHAR(20) NOT NULL,
  email VARCHAR(100) NOT NULL
);

GO
CREATE TABLE Pessoas_Físicas (
  id INT PRIMARY KEY,
  cpf VARCHAR(11) NOT NULL,
  data_nascimento DATE NOT NULL,
  FOREIGN KEY (id) REFERENCES Pessoas(id)
);

GO
CREATE TABLE Pessoas_Jurídicas (
  id INT PRIMARY KEY,
  cnpj VARCHAR(14) NOT NULL,
  razão_social VARCHAR(100) NOT NULL,
  FOREIGN KEY (id) REFERENCES Pessoas(id)
);

GO
CREATE TABLE Produtos (
  id INT PRIMARY KEY IDENTITY(1,1),
  nome VARCHAR(100) NOT NULL,
  quantidade INT NOT NULL,
  preço DECIMAL(10, 2) NOT NULL
);

GO
CREATE TABLE Compras (
  id INT PRIMARY KEY IDENTITY(1,1),
  user_id INT NOT NULL,
  product_id INT NOT NULL,
  juridical_person_id INT NOT NULL,
  quantidade INT NOT NULL,
  preço_unitário DECIMAL(10, 2) NOT NULL,
  data_de_compra DATE NOT NULL,
  FOREIGN KEY (user_id) REFERENCES Usuarios(id),
  FOREIGN KEY (product_id) REFERENCES Produtos(id),
  FOREIGN KEY (juridical_person_id) REFERENCES Pessoas(id)
);

GO
CREATE TABLE Vendas (
  id INT PRIMARY KEY IDENTITY(1,1),
  user_id INT NOT NULL,
  product_id INT NOT NULL,
  physical_person_id INT NOT NULL,
  quantidade INT NOT NULL,
  preço_unitário DECIMAL(10, 2) NOT NULL,
  data_de_venda DATE NOT NULL,
  FOREIGN KEY (user_id) REFERENCES Usuarios(id),
  FOREIGN KEY (product_id) REFERENCES Produtos(id),
  FOREIGN KEY (physical_person_id) REFERENCES Pessoas(id)
);

-- Criar trigger para inserir default values into Pessoas table
GO
CREATE TRIGGER trg_Pessoas_Insert
ON Pessoas
INSTEAD OF INSERT
AS
BEGIN
    DECLARE @id INT;
    EXEC dbo.GetNextPessoaId @id OUTPUT;
    INSERT INTO Pessoas (id, tipo, nome, identificação, endereço, telefone, email)
    SELECT @id, i.tipo, i.nome, i.identificação, i.endereço, i.telefone, i.email
    FROM inserted i;
END;
GO
