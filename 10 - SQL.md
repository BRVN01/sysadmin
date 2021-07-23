[toc]



# Introdução

Trabalhar com manipulação de dados num banco de dados, isso não visa tornar nenhum dba, apenas mostra conceitos de como trabalhar com databases, fazer migração, criação de tabelas, apenas o básico.





## Instalação

Vamos instalar o MariaDB, um sistema de banco de dados relacional, derivado do MySQL.

```bash
## Instalar o pacote mariadb-server:
$ apt-get install mariadb-server 
# ou "yum install mariadb-server" caso esteja utilizando um sistema baseado em RedHat

## Configurar o MariaDB:
$ sudo mysql_secure_installation

Set root password? [Y/n] n
Remove anonymous users? [Y/n] n
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] n
Reload privilege tables now? [Y/n] Y
# Estas foram as opções que eu escolhi para tornar o uso o mais fácil possível. Se quiser definir senhas e outras configurações, fique a vontade.
```



### Instalar um Banco de Dados de Exemplo

Para facilitar os estudos, vamos fazer o download e importar para o banco uma base de dados e tabelas já criadas que podemos encontrar no GitHub.

```bash
# Baixa o db:
wget https://github.com/datacharmer/test_db/archive/master.zip

# Descompacte:
unzip master.zip

# Entre na pasta:
cd test_db-master

# Importe o db:
mysql -u root < employees.sql

# Listando os databases sem entrar no mysql (mariadb):
$ sudo mysqlshow 
+--------------------+
|     Databases      |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```





## Trabalhando com o banco

```bash
# Acessando o db (Caso tenha colocado senha, coloque a opção '-p'):
$ sudo mysql -u root

# Entre no banco chamado emplyees:
MariaDB [(none)]> use employees;
Database changed

# Liste as tables existentes dentro do nosso banco de dados:
MariaDB [employees]> show tables;
+----------------------+
| Tables_in_employees  |
+----------------------+
| current_dept_emp     |
| departments          |
| dept_emp             |
| dept_emp_latest_date |
| dept_manager         |
| employees            |
| salaries             |
| titles               |
+----------------------+
8 rows in set (0.001 sec)

# Liste a estrutura da tabela, assim podemos listar dados palpáveis dentro dela:
MariaDB [employees]> describe departments;
+-----------+-------------+------+-----+---------+-------+
| Field     | Type        | Null | Key | Default | Extra |
+-----------+-------------+------+-----+---------+-------+
| dept_no   | char(4)     | NO   | PRI | NULL    |       |
| dept_name | varchar(40) | NO   | UNI | NULL    |       |
+-----------+-------------+------+-----+---------+-------+
2 rows in set (0.003 sec)

# Liste algumas informações:
MariaDB [employees]> select dept_no,dept_name from departments;
+---------+--------------------+
| dept_no | dept_name          |
+---------+--------------------+
| d009    | Customer Service   |
| d005    | Development        |
| d002    | Finance            |
| d003    | Human Resources    |
| d001    | Marketing          |
| d004    | Production         |
| d006    | Quality Management |
| d008    | Research           |
| d007    | Sales              |
+---------+--------------------+
9 rows in set (0.002 sec)
# Estrutura: select CAMPOS from TABELA;
# Você pode mudar colocar um '*' no lugar do nome do campo, assim ele vai mostrar tudo.

# Limitando a saída:
MariaDB [employees]> select dept_no,dept_name from departments limit 3;
+---------+------------------+
| dept_no | dept_name        |
+---------+------------------+
| d009    | Customer Service |
| d005    | Development      |
| d002    | Finance          |
+---------+------------------+
3 rows in set (0.002 sec)

# Contando a quantidade de registros dentro de uma tabela:
MariaDB [employees]> select count(*) from employees;
+----------+
| count(*) |
+----------+
|   300024 |
+----------+
1 row in set (0.078 sec)
# Isso significa que temos mais de 300k de linhas nessa tabela.

# Criando um filtro com WHERE, listando apenas funcionarios contratados depois dos anos 2k:
MariaDB [employees]> select * from employees where hire_date > "2000-01-01 limit 10";
+--------+------------+------------+------------+--------+------------+
| emp_no | birth_date | first_name | last_name  | gender | hire_date  |
+--------+------------+------------+------------+--------+------------+
|  47291 | 1960-09-09 | Ulf        | Flexer     | M      | 2000-01-12 |
|  60134 | 1964-04-21 | Seshu      | Rathonyi   | F      | 2000-01-02 |
|  72329 | 1953-02-09 | Randi      | Luit       | F      | 2000-01-02 |
| 205048 | 1960-09-12 | Ennio      | Alblas     | F      | 2000-01-06 |
| 222965 | 1959-08-07 | Volkmar    | Perko      | F      | 2000-01-13 |
| 226633 | 1958-06-10 | Xuejun     | Benzmuller | F      | 2000-01-04 |
| 227544 | 1954-11-17 | Shahab     | Demeyer    | M      | 2000-01-08 |
| 422990 | 1953-04-09 | Jaana      | Verspoor   | F      | 2000-01-11 |
| 424445 | 1953-04-27 | Jeong      | Boreale    | M      | 2000-01-03 |
| 428377 | 1957-05-09 | Yucai      | Gerlach    | M      | 2000-01-23 |
| 463807 | 1964-06-12 | Bikash     | Covnot     | M      | 2000-01-28 |
| 499553 | 1954-05-06 | Hideyuki   | Delgrande  | F      | 2000-01-22 |
+--------+------------+------------+------------+--------+------------+
12 rows in set, 1 warning (0.121 sec)

# Funcionarios contratados depois dos anos 2k e que sejam mulheres:
MariaDB [employees]> select * from employees where hire_date > "2000-01-01" AND gender="F";
+--------+------------+------------+------------+--------+------------+
| emp_no | birth_date | first_name | last_name  | gender | hire_date  |
+--------+------------+------------+------------+--------+------------+
|  60134 | 1964-04-21 | Seshu      | Rathonyi   | F      | 2000-01-02 |
|  72329 | 1953-02-09 | Randi      | Luit       | F      | 2000-01-02 |
| 205048 | 1960-09-12 | Ennio      | Alblas     | F      | 2000-01-06 |
| 222965 | 1959-08-07 | Volkmar    | Perko      | F      | 2000-01-13 |
| 226633 | 1958-06-10 | Xuejun     | Benzmuller | F      | 2000-01-04 |
| 422990 | 1953-04-09 | Jaana      | Verspoor   | F      | 2000-01-11 |
| 499553 | 1954-05-06 | Hideyuki   | Delgrande  | F      | 2000-01-22 |
+--------+------------+------------+------------+--------+------------+
7 rows in set (0.118 sec)

# Mesma coisa acima, mas aqui, vamos listar apenas as pessoas que começam com a letra 'S':
MariaDB [employees]> select * from employees where hire_date > "2000-01-01" and first_name like "S%";
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
|  60134 | 1964-04-21 | Seshu      | Rathonyi  | F      | 2000-01-02 |
| 227544 | 1954-11-17 | Shahab     | Demeyer   | M      | 2000-01-08 |
+--------+------------+------------+-----------+--------+------------+
2 rows in set (0.109 sec)

# Mesma coisa acima, mas aqui, vamos listar apenas as pessoas que começam com a letra 'S' ou 'P':
MariaDB [employees]> select * from employees where hire_date > "2000-01-01" and (first_name like "S%" OR first_name like "P%");

```

