#DESAFIO DIO MYSQL
---

## **Parte 1 – Criação de Views e Definição de Permissões**

### **1. Views Criadas para os Cenários**

1. **Número de empregados por departamento e localidade**:
   ```sql
   CREATE VIEW vw_empregados_por_departamento_localidade AS
   SELECT D.Dname AS departamento, L.Dlocation AS localidade, COUNT(E.Ssn) AS total_empregados
   FROM Employee E
   JOIN Department D ON E.Dno = D.Dnumber
   JOIN Dept_Locations L ON D.Dnumber = L.Dnumber
   GROUP BY D.Dname, L.Dlocation;
   ```

2. **Lista de departamentos e seus gerentes**:
   ```sql
   CREATE VIEW vw_departamentos_gerentes AS
   SELECT D.Dname AS departamento, E.Fname AS gerente_nome, E.Lname AS gerente_sobrenome
   FROM Department D
   JOIN Employee E ON D.Mgr_ssn = E.Ssn;
   ```

3. **Projetos com maior número de empregados (ordenado por ordem decrescente)**:
   ```sql
   CREATE VIEW vw_projetos_mais_empregados AS
   SELECT P.Pname AS projeto, COUNT(W.Essn) AS total_empregados
   FROM Project P
   JOIN Works_On W ON P.Pnumber = W.Pno
   GROUP BY P.Pname
   ORDER BY total_empregados DESC;
   ```

4. **Lista de projetos, departamentos e gerentes**:
   ```sql
   CREATE VIEW vw_projetos_departamentos_gerentes AS
   SELECT P.Pname AS projeto, D.Dname AS departamento, E.Fname AS gerente_nome, E.Lname AS gerente_sobrenome
   FROM Project P
   JOIN Department D ON P.Dnum = D.Dnumber
   JOIN Employee E ON D.Mgr_ssn = E.Ssn;
   ```

5. **Quais empregados possuem dependentes e se são gerentes**:
   ```sql
   CREATE VIEW vw_empregados_com_dependentes_e_gerencia AS
   SELECT E.Fname AS empregado_nome, E.Lname AS empregado_sobrenome, 
          CASE WHEN E.Ssn = D.Mgr_ssn THEN 'Sim' ELSE 'Não' END AS gerente
   FROM Employee E
   JOIN Dependent Dp ON E.Ssn = Dp.Essn
   LEFT JOIN Department D ON E.Ssn = D.Mgr_ssn;
   ```

---

### **2. Definição de Permissões de Acesso às Views**

1. **Criação dos Usuários e Permissões**:

   - **Usuário gerente**:  
     Pode acessar as informações de `Employee` e `Department`.
     ```sql
     CREATE USER 'gerente'@'localhost' IDENTIFIED BY 'senha123';
     GRANT SELECT ON company.Employee TO 'gerente'@'localhost';
     GRANT SELECT ON company.Department TO 'gerente'@'localhost';
     ```

   - **Usuário empregado**:  
     Não tem acesso às informações de `Department` ou gerentes.
     ```sql
     CREATE USER 'empregado'@'localhost' IDENTIFIED BY 'senha123';
     GRANT SELECT ON company.Employee TO 'empregado'@'localhost';
     ```

---

## **Parte 2 – Criação de Triggers para o Cenário de E-commerce**

### **1. Trigger para Remoção: Before Delete**

Quando um usuário é excluído, suas informações são registradas em uma tabela de histórico para referência futura.

```sql
CREATE TABLE Historico_Usuario (
    idUsuario INT,
    tipo VARCHAR(20),
    nome VARCHAR(100),
    data_remocao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TRIGGER before_delete_cliente
BEFORE DELETE ON Cliente
FOR EACH ROW
BEGIN
    INSERT INTO Historico_Usuario (idUsuario, tipo, nome) 
    VALUES (OLD.idCliente, 'Cliente', OLD.nome);
END;
```

---

### **2. Trigger para Atualização: Before Update**

Este gatilho verifica se houve aumento salarial ao atualizar um colaborador e registra a alteração em uma tabela de auditoria.

```sql
CREATE TABLE Auditoria_Salario (
    idColaborador INT,
    salario_antigo DECIMAL(10, 2),
    salario_novo DECIMAL(10, 2),
    data_alteracao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TRIGGER before_update_employee
BEFORE UPDATE ON Employee
FOR EACH ROW
BEGIN
    IF OLD.Salary <> NEW.Salary THEN
        INSERT INTO Auditoria_Salario (idColaborador, salario_antigo, salario_novo)
        VALUES (OLD.Ssn, OLD.Salary, NEW.Salary);
    END IF;
END;
```

---
