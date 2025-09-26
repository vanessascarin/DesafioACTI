-- Criar um banco de dados para um sistema simples de bibioteca.
CREATE DATABASE [Biblioteca-DesafioACTI];
GO

USE [Biblioteca-DesafioACTI];
GO

-- Cria a tabela de Livro
-- Status será disponível como padrão
-- Permite várias cópias do mesmo livro
CREATE TABLE Livro (
    id_livro INT PRIMARY KEY IDENTITY,
    titulo VARCHAR(255) NOT NULL,
    autor VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'DISPONIVEL'
);
GO

-- Cria a tabela de Leitor
-- Permite vários leitores com o mesmo telefone
CREATE TABLE Leitor (
    id_leitor INT PRIMARY KEY IDENTITY,
    nome VARCHAR(255) NOT NULL UNIQUE,
    telefone VARCHAR(20)
);
GO

-- Cria a tabela de Empréstimo
-- Permite somente um empréstimo por livro e por leitor
CREATE TABLE Emprestimo (
    id_emprestimo INT PRIMARY KEY IDENTITY,
    id_livro INT FOREIGN KEY REFERENCES Livro(id_livro) UNIQUE,
    id_leitor INT FOREIGN KEY REFERENCES Leitor(id_leitor) UNIQUE,
    data_emprestimo DATETIME NOT NULL,
    data_devolucao DATETIME NOT NULL
);
GO

-- CRUD DE LIVRO
-- Stored procedure para inserir um novo livro (CREATE)
CREATE PROCEDURE sp_Livro_Insert
    @titulo VARCHAR(255),
    @autor VARCHAR(255)
AS
BEGIN
    INSERT INTO Livro (titulo, autor)
    VALUES (@titulo, @autor);
END;
GO

-- Stored procedure para consultar livros (READ)
CREATE PROCEDURE sp_Livro_Select
    @id_livro INT = NULL
AS
BEGIN
    IF @id_livro IS NULL
    BEGIN
        SELECT * FROM Livro;
    END
    ELSE
    BEGIN
        SELECT * FROM Livro WHERE id_livro = @id_livro;
    END
END;
GO

-- Stored procedure para atualizar um livro (UPDATE)
CREATE PROCEDURE sp_Livro_Update
    @id_livro INT,
    @titulo VARCHAR(255) = NULL,
    @autor VARCHAR(255) = NULL
AS
BEGIN
    UPDATE Livro
    SET
        titulo = ISNULL(@titulo, titulo),
        autor = ISNULL(@autor, autor)
    WHERE id_livro = @id_livro;
END;
GO

-- Stored procedure para excluir um livro (DELETE)
-- Somente livros sem pendências podem ser excluídos
CREATE PROCEDURE sp_Livro_Delete
    @id_livro INT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Emprestimo WHERE id_livro = @id_livro)
    BEGIN
        RAISERROR('Não é possível excluir o livro, pois ele está emprestado.', 16, 1);
        RETURN;
    END

    DELETE FROM Livro WHERE id_livro = @id_livro;
END;
GO

-- CRUD DE LEITOR
-- Stored procedure para inserir um novo Leitor (CREATE) 
-- Impede cadastros duplicados pelo nome
CREATE PROCEDURE sp_Leitor_Insert
    @nome VARCHAR(255),
    @telefone VARCHAR(20)
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Leitor WHERE nome = @nome)
    BEGIN
        RAISERROR('Este leitor já possui cadastro!', 16, 1);
        RETURN;
    END

    INSERT INTO Leitor (nome, telefone)
    VALUES (@nome, @telefone);
END;
GO

-- Stored procedure para consultar leitores (READ)
CREATE PROCEDURE sp_Leitor_Select
    @id_leitor INT = NULL
AS
BEGIN
    IF @id_leitor IS NULL
    BEGIN
        SELECT * FROM Leitor;
    END
    ELSE
    BEGIN
        SELECT * FROM Leitor WHERE id_leitor = @id_leitor;
    END
END;
GO

-- Stored procedure para atualizar um Leitor (UPDATE)
CREATE PROCEDURE sp_Leitor_Update
    @id_leitor INT,
    @nome VARCHAR(255) = NULL,
    @telefone VARCHAR(20) = NULL
AS
BEGIN
    IF @nome IS NOT NULL AND EXISTS (SELECT 1 FROM Leitor WHERE nome = @nome AND id_leitor <> @id_leitor)
    BEGIN
        RAISERROR('Já existe um leitor com este nome.', 16, 1);
        RETURN;
    END

    UPDATE Leitor
    SET
        nome = ISNULL(@nome, nome),
        telefone = ISNULL(@telefone, telefone)
    WHERE id_leitor = @id_leitor;
END;
GO

-- Stored procedure para excluir um Leitor (DELETE)
-- Somente leitores sem pendências podem ser excluídos
CREATE PROCEDURE sp_Leitor_Delete
    @id_leitor INT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Emprestimo WHERE id_leitor = @id_leitor)
    BEGIN
        RAISERROR('Não é possível excluir o leitor pois ele possui um empréstimo ativo!', 16, 1);
        RETURN;
    END

    DELETE FROM Leitor WHERE id_leitor = @id_leitor;
END;
GO

-- GESTÃO DE EMPRÉSTIMOS
-- Stored procedure para registrar um novo empréstimo
CREATE PROCEDURE sp_Emprestimo_Emprestar
    @id_livro INT,
    @id_leitor INT
AS
BEGIN
    -- Verifica se o leitor já tem um empréstimo ativo
    IF EXISTS (SELECT 1 FROM Emprestimo WHERE id_leitor = @id_leitor)
    BEGIN
        RAISERROR('O leitor já possui um empréstimo ativo e não pode emprestar outro livro.', 16, 1);
        RETURN;
    END

    -- Verifica se o livro está disponível
    IF (SELECT status FROM Livro WHERE id_livro = @id_livro) = 'EMPRESTADO'
    BEGIN
        -- Busca a data de devolução 
        DECLARE @data_devolucao DATETIME;
        SELECT @data_devolucao = data_devolucao
        FROM Emprestimo
        WHERE id_livro = @id_livro;

		--Formata a data de devolução
		DECLARE @data_formatada VARCHAR(10);
        SET @data_formatada = CONVERT(VARCHAR, @data_devolucao, 103);

        RAISERROR('O livro já está emprestado e deve ser devolvido em %s.', 16, 1, @data_formatada);
        RETURN;
    END

    DECLARE @data_inicio DATETIME = GETDATE();
    DECLARE @data_fim DATETIME = DATEADD(day, 7, @data_inicio);

    INSERT INTO Emprestimo (id_livro, id_leitor, data_emprestimo, data_devolucao)
    VALUES (@id_livro, @id_leitor, @data_inicio, @data_fim);

    UPDATE Livro SET status = 'EMPRESTADO' WHERE id_livro = @id_livro;
END;
GO

-- Stored procedure para devolver um livro
CREATE PROCEDURE sp_Emprestimo_Devolver
    @id_livro INT
AS
BEGIN
    -- Deleta o registro de empréstimo
    DELETE FROM Emprestimo WHERE id_livro = @id_livro;

    -- Atualiza o status do livro para 'DISPONIVEL'
    UPDATE Livro SET status = 'DISPONIVEL' WHERE id_livro = @id_livro;
END;
GO
