-- Tabela: Clientes
CREATE TABLE Clientes (
    ClienteID INT PRIMARY KEY AUTO_INCREMENT,
    Nome VARCHAR(255) NOT NULL,
    Telefone VARCHAR(20),
    Endereco VARCHAR(255)
);

-- Tabela: Veiculos
CREATE TABLE Veiculos (
    VeiculoID INT PRIMARY KEY AUTO_INCREMENT,
    ClienteID INT NOT NULL,
    Placa VARCHAR(10) UNIQUE NOT NULL,
    Modelo VARCHAR(100) NOT NULL,
    Ano INT NOT NULL,
    FOREIGN KEY (ClienteID) REFERENCES Clientes(ClienteID)
);

-- Tabela: Servicos
CREATE TABLE Servicos (
    ServicoID INT PRIMARY KEY AUTO_INCREMENT,
    Descricao VARCHAR(255) NOT NULL,
    Preco DECIMAL(10, 2) NOT NULL
);

-- Tabela: OrdensServico
CREATE TABLE OrdensServico (
    OrdemID INT PRIMARY KEY AUTO_INCREMENT,
    VeiculoID INT NOT NULL,
    DataAbertura DATE NOT NULL,
    DataFechamento DATE,
    Status ENUM('Aberta', 'Fechada', 'Cancelada') NOT NULL,
    FOREIGN KEY (VeiculoID) REFERENCES Veiculos(VeiculoID)
);

-- Tabela: Ordens_Servicos (Relacionamento N:N entre Ordens e Serviços)
CREATE TABLE Ordens_Servicos (
    OrdemID INT NOT NULL,
    ServicoID INT NOT NULL,
    Quantidade INT NOT NULL,
    PRIMARY KEY (OrdemID, ServicoID),
    FOREIGN KEY (OrdemID) REFERENCES OrdensServico(OrdemID),
    FOREIGN KEY (ServicoID) REFERENCES Servicos(ServicoID)
);

-- Inserção de dados para teste

-- Clientes
INSERT INTO Clientes (Nome, Telefone, Endereco) VALUES
('Carlos Silva', '11987654321', 'Rua A, 123'),
('Ana Oliveira', '11976543210', 'Rua B, 456');

-- Veículos
INSERT INTO Veiculos (ClienteID, Placa, Modelo, Ano) VALUES
(1, 'ABC1234', 'Fiat Uno', 2010),
(2, 'XYZ5678', 'Honda Civic', 2020);

-- Serviços
INSERT INTO Servicos (Descricao, Preco) VALUES
('Troca de óleo', 150.00),
('Alinhamento e balanceamento', 200.00),
('Revisão completa', 500.00);

-- Ordens de Serviço
INSERT INTO OrdensServico (VeiculoID, DataAbertura, Status) VALUES
(1, '2024-01-01', 'Aberta'),
(2, '2024-01-02', 'Fechada');

-- Relacionamento Ordens_Servicos
INSERT INTO Ordens_Servicos (OrdemID, ServicoID, Quantidade) VALUES
(1, 1, 1),
(1, 2, 1),
(2, 3, 1);

-- Consultas complexas

-- 1. Lista de serviços realizados em cada veículo
SELECT v.Placa, s.Descricao, os.Quantidade
FROM Veiculos v
INNER JOIN OrdensServico o ON v.VeiculoID = o.VeiculoID
INNER JOIN Ordens_Servicos os ON o.OrdemID = os.OrdemID
INNER JOIN Servicos s ON os.ServicoID = s.ServicoID;

-- 2. Total gasto por cada cliente
SELECT c.Nome AS Cliente, SUM(s.Preco * os.Quantidade) AS TotalGasto
FROM Clientes c
INNER JOIN Veiculos v ON c.ClienteID = v.ClienteID
INNER JOIN OrdensServico o ON v.VeiculoID = o.VeiculoID
INNER JOIN Ordens_Servicos os ON o.OrdemID = os.OrdemID
INNER JOIN Servicos s ON os.ServicoID = s.ServicoID
GROUP BY c.Nome;

-- 3. Ordens de serviço abertas ordenadas por data de abertura
SELECT o.OrdemID, v.Placa, o.DataAbertura
FROM OrdensServico o
INNER JOIN Veiculos v ON o.VeiculoID = v.VeiculoID
WHERE o.Status = 'Aberta'
ORDER BY o.DataAbertura;

-- 4. Quantidade de serviços realizados por veículo
SELECT v.Placa, COUNT(os.ServicoID) AS TotalServicos
FROM Veiculos v
INNER JOIN OrdensServico o ON v.VeiculoID = o.VeiculoID
INNER JOIN Ordens_Servicos os ON o.OrdemID = os.OrdemID
GROUP BY v.Placa;

-- 5. Clientes com ordens de serviço fechadas e total gasto
SELECT c.Nome AS Cliente, SUM(s.Preco * os.Quantidade) AS TotalGasto
FROM Clientes c
INNER JOIN Veiculos v ON c.ClienteID = v.ClienteID
INNER JOIN OrdensServico o ON v.VeiculoID = o.VeiculoID
INNER JOIN Ordens_Servicos os ON o.OrdemID = os.OrdemID
INNER JOIN Servicos s ON os.ServicoID = s.ServicoID
WHERE o.Status = 'Fechada'
GROUP BY c.Nome;

-- 6. Serviços mais realizados e sua quantidade
SELECT s.Descricao, COUNT(os.ServicoID) AS TotalRealizado
FROM Servicos s
INNER JOIN Ordens_Servicos os ON s.ServicoID = os.ServicoID
GROUP BY s.Descricao
ORDER BY TotalRealizado DESC;

