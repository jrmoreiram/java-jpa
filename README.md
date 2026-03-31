# 📚 Sistema de Loja com Java e JPA

## 📋 Sobre o Projeto

Este projeto é uma aplicação Java que demonstra o uso avançado de **JPA (Java Persistence API)** e **Hibernate** para gerenciamento de persistência de dados em um sistema de loja. O projeto foi desenvolvido com base no curso **"Java e JPA: Consultas Avançadas, Performance e Modelos Complexos"** da **DevMedia**, explorando técnicas sofisticadas de mapeamento objeto-relacional e otimização de consultas.

## 🎯 Objetivo

Demonstrar na prática conceitos avançados de JPA através de um sistema de loja completo, incluindo:

- Gerenciamento de produtos e categorias
- Controle de pedidos e itens de pedido
- Cadastro de clientes
- Relatórios de vendas
- Consultas otimizadas com foco em performance

## 🛠️ Tecnologias Utilizadas

### Core
- **Java 11** - Linguagem de programação principal
- **Maven** - Gerenciador de dependências e build
- **JPA 2.2** - API de persistência Java

### Frameworks e Bibliotecas
- **Hibernate 5.4.27.Final** - Implementação JPA e ORM (Object-Relational Mapping)
- **H2 Database 1.4.200** - Banco de dados em memória para testes e desenvolvimento

### Plugins
- **Maven Compiler Plugin 3.8.0** - Compilação do projeto com Java 11

## 🏗️ Arquitetura do Projeto

### Estrutura de Pacotes

```
br.com.alura.loja
│
├── dao/                    # Data Access Objects (Camada de Persistência)
│   ├── CategoriaDao.java
│   ├── ClienteDao.java
│   ├── PedidoDao.java
│   └── ProdutoDao.java
│
├── modelo/                 # Entidades JPA (Modelo de Domínio)
│   ├── Categoria.java
│   ├── CategoriaId.java    # Chave Composta
│   ├── Cliente.java
│   ├── DadosPessoais.java  # Classe Embeddable
│   ├── Informatica.java    # Herança
│   ├── ItemPedido.java
│   ├── Livro.java          # Herança
│   ├── Pedido.java
│   └── Produto.java
│
├── testes/                 # Classes de Teste e Demonstração
│   ├── CadastroDePedido.java
│   ├── CadastroDeProduto.java
│   ├── PerformanceConsultas.java
│   └── TesteCriteria.java
│
├── util/                   # Utilitários
│   └── JPAUtil.java        # Gerenciador de EntityManager
│
└── vo/                     # Value Objects
    └── RelatorioDeVendasVo.java
```

## 🔑 Recursos e Funcionalidades Implementadas

### 1. **Mapeamento de Entidades**

#### Relacionamentos Bidirecionais
- `@OneToMany` e `@ManyToOne` entre Pedido ↔ ItemPedido
- `@ManyToOne` entre Produto → Categoria
- `@ManyToOne` entre Pedido → Cliente

#### Herança de Entidades
- Estratégia `JOINED` para herança de Produto
- Entidades especializadas: `Livro` e `Informatica`
- Permite polimorfismo e consultas polimórficas

#### Chave Composta (@EmbeddedId)
- Classe `CategoriaId` como chave composta
- Implementa `Serializable` conforme especificação JPA

#### Classe Embeddable (@Embedded)
- `DadosPessoais` embutida na entidade Cliente
- Reutilização de atributos comuns

### 2. **Estratégias de Carregamento**

#### Lazy Loading (Carregamento Preguiçoso)
```java
@ManyToOne(fetch = FetchType.LAZY)
private Categoria categoria;
```
- Evita carregamento desnecessário de dados
- Melhora performance inicial

#### Eager Loading com JOIN FETCH
```java
SELECT p FROM Pedido p JOIN FETCH p.cliente WHERE p.id = :id
```
- Resolve problema N+1
- Carrega dados relacionados em uma única query

### 3. **Consultas Avançadas JPQL**

#### Named Queries
```java
@NamedQuery(name = "Produto.produtosPorCategoria", 
    query = "SELECT p FROM Produto p WHERE p.categoria.id.nome = :nome")
```

#### Funções de Agregação
- `SUM()` - Cálculo de valor total vendido
- `MAX()` - Data da última venda
- `GROUP BY` - Agrupamento para relatórios

#### SELECT NEW (Constructor Expression)
```java
SELECT new br.com.alura.loja.vo.RelatorioDeVendasVo(
    produto.nome, 
    SUM(item.quantidade), 
    MAX(pedido.data)
) FROM Pedido pedido ...
```
- Retorna DTOs diretamente das consultas
- Reduz tráfego de dados desnecessários

#### Consultas com Joins
```java
FROM Pedido pedido 
JOIN pedido.itens item 
JOIN item.produto produto
```

### 4. **Criteria API**

Consultas dinâmicas com type-safety:

```java
CriteriaBuilder builder = em.getCriteriaBuilder();
CriteriaQuery<Produto> query = builder.createQuery(Produto.class);
Root<Produto> from = query.from(Produto.class);

Predicate filtros = builder.and();
if (nome != null) {
    filtros = builder.and(filtros, builder.equal(from.get("nome"), nome));
}
query.where(filtros);
```

**Vantagens:**
- Type-safe em tempo de compilação
- Consultas dinâmicas baseadas em parâmetros opcionais
- Refatoração mais segura

### 5. **Padrão DAO (Data Access Object)**

Encapsulamento das operações de persistência:
- `cadastrar()` - Persist
- `atualizar()` - Merge
- `remover()` - Remove
- `buscarPorId()` - Find
- Métodos de consulta customizados

### 6. **Cascade e Relacionamentos**

```java
@OneToMany(mappedBy = "pedido", cascade = CascadeType.ALL)
private List<ItemPedido> itens = new ArrayList<>();
```

- Cascata de operações para itens do pedido
- Simplifica gerenciamento de entidades relacionadas

### 7. **Relatórios e Agregações**

#### Relatório de Vendas
- Produto mais vendido
- Quantidade total por produto
- Data da última venda
- Ordenação por quantidade

## ⚙️ Configuração do Projeto

### persistence.xml

```xml
<persistence-unit name="loja" transaction-type="RESOURCE_LOCAL">
    <properties>
        <!-- Conexão H2 em memória -->
        <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
        <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:loja"/>
        
        <!-- Configurações do Hibernate -->
        <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
        <property name="hibernate.show_sql" value="true"/>
        <property name="hibernate.format_sql" value="true"/>
        <property name="hibernate.hbm2ddl.auto" value="update"/>
    </properties>
</persistence-unit>
```

**Características:**
- Banco de dados H2 em memória
- SQL visível no console (show_sql)
- SQL formatado para melhor leitura
- Auto-criação/atualização de schema

## 🚀 Como Executar

### Pré-requisitos
- JDK 11 ou superior
- Maven 3.x

### Passos

1. **Clone ou extraia o projeto**
```bash
cd java-jpa-main
```

2. **Compile o projeto**
```bash
mvn clean compile
```

3. **Execute uma classe de teste**
```bash
mvn exec:java -Dexec.mainClass="br.com.alura.loja.testes.CadastroDePedido"
```

ou

```bash
mvn exec:java -Dexec.mainClass="br.com.alura.loja.testes.PerformanceConsultas"
```

ou

```bash
mvn exec:java -Dexec.mainClass="br.com.alura.loja.testes.TesteCriteria"
```

## 📊 Modelo de Dados

### Entidades Principais

| Entidade | Descrição | Relacionamentos |
|----------|-----------|-----------------|
| **Produto** | Produtos da loja (classe abstrata) | ManyToOne com Categoria |
| **Livro** | Especialização de Produto | Herda de Produto |
| **Informatica** | Especialização de Produto | Herda de Produto |
| **Categoria** | Categorias de produtos | OneToMany com Produto |
| **Cliente** | Clientes da loja | OneToMany com Pedido |
| **Pedido** | Pedidos realizados | ManyToOne com Cliente, OneToMany com ItemPedido |
| **ItemPedido** | Itens de um pedido | ManyToOne com Pedido e Produto |

### Diagrama de Relacionamentos

```
Categoria (1) ──────< (*) Produto
                            │
                            ├─── Livro
                            └─── Informatica

Cliente (1) ──────< (*) Pedido (1) ──────< (*) ItemPedido (*) >────── (1) Produto
```

## 🎓 Conceitos de JPA Demonstrados

### ✅ Relacionamentos
- [x] @OneToMany
- [x] @ManyToOne
- [x] Bidirecionalidade com `mappedBy`

### ✅ Estratégias de Fetch
- [x] FetchType.LAZY
- [x] FetchType.EAGER
- [x] JOIN FETCH

### ✅ Herança
- [x] @Inheritance
- [x] InheritanceType.JOINED

### ✅ Chaves
- [x] @Id com @GeneratedValue
- [x] @EmbeddedId (Chave Composta)

### ✅ Consultas
- [x] JPQL
- [x] @NamedQuery
- [x] SELECT NEW
- [x] Criteria API
- [x] Funções de Agregação

### ✅ Cascade e Performance
- [x] CascadeType.ALL
- [x] Consultas otimizadas
- [x] Projeções com VO

## 📝 Exemplos de Código

### Consulta com SELECT NEW
```java
String jpql = "SELECT new br.com.alura.loja.vo.RelatorioDeVendasVo(" +
              "produto.nome, " +
              "SUM(item.quantidade), " +
              "MAX(pedido.data)) " +
              "FROM Pedido pedido " +
              "JOIN pedido.itens item " +
              "JOIN item.produto produto " +
              "GROUP BY produto.nome " +
              "ORDER BY item.quantidade DESC";
              
return em.createQuery(jpql, RelatorioDeVendasVo.class).getResultList();
```

### Criteria API Dinâmica
```java
public List<Produto> buscarPorParametrosComCriteria(String nome, 
        BigDecimal preco, LocalDate dataCadastro) {

    CriteriaBuilder builder = em.getCriteriaBuilder();
    CriteriaQuery<Produto> query = builder.createQuery(Produto.class);
    Root<Produto> from = query.from(Produto.class);
    
    Predicate filtros = builder.and();
    if (nome != null && !nome.trim().isEmpty()) {
        filtros = builder.and(filtros, builder.equal(from.get("nome"), nome));
    }
    if (preco != null) {
        filtros = builder.and(filtros, builder.equal(from.get("preco"), preco));
    }
    if (dataCadastro != null) {
        filtros = builder.and(filtros, builder.equal(from.get("dataCadastro"), dataCadastro));
    }
    query.where(filtros);
    
    return em.createQuery(query).getResultList();
}
```

## 🔍 Problemas de Performance Resolvidos

### Problema N+1
**Antes:** Múltiplas queries para carregar dados relacionados
```java
Pedido pedido = em.find(Pedido.class, 1L); // 1 query
pedido.getCliente().getNome(); // +1 query adicional!
```

**Depois:** JOIN FETCH resolve em uma única query
```java
SELECT p FROM Pedido p JOIN FETCH p.cliente WHERE p.id = :id
```

### Lazy vs Eager
- Uso estratégico de `FetchType.LAZY` para evitar carregamento desnecessário
- JOIN FETCH quando dados relacionados são necessários

## 📚 Referências

Este projeto foi desenvolvido com base no curso:
**"Java e JPA: Consultas Avançadas, Performance e Modelos Complexos"** - DevMedia

### Tópicos Abordados no Curso:
- ✅ Modelagem de relacionamentos bidirecionais
- ✅ SELECT NEW para consultas avançadas
- ✅ Diferença entre EAGER e LAZY
- ✅ JOIN FETCH para planejamento de queries
- ✅ API de Criteria da JPA
- ✅ Mapeamento de herança
- ✅ Chaves compostas

## 📄 Licença

Este é um projeto educacional desenvolvido para fins de aprendizado.

## 👨‍💻 Autor

Projeto desenvolvido durante o curso de Java e JPA da DevMedia.

---

**Nota:** Este README técnico foi criado para servir como documentação completa do projeto, detalhando arquitetura, recursos, tecnologias e conceitos aplicados.
