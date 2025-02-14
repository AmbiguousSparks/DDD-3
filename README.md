# 📚 Trabalho - Design Tático no DDD

## 📌 Objetivo
Criar as **Entidades, Value Objects, Agregados e Repositórios** do seu projeto, aplicando o **Design Tático do DDD**.

---

## 📂 **1️⃣ Conceitos Fundamentais**

### **🧩 Entidades vs. Objetos de Valor**
📌 **Entidades** têm **identidade única** e podem mudar ao longo do tempo.  
📌 **Objetos de Valor** não possuem identidade própria e são **imutáveis**.  

💡 **Exemplo no Contexto de Consultas Médicas:**  

| **Elemento**      | **Entidade ou Value Object?** | **Justificativa** |
|------------------|-----------------------------|-------------------|
| **Paciente**     | Entidade                     | Tem um ID único e pode mudar seus dados ao longo do tempo. |
| **Médico**       | Entidade                     | Possui identidade única e pode atualizar sua especialidade. |
| **Endereço**     | Value Object                 | Se um paciente muda de endereço, não faz sentido modificar o antigo, apenas substituí-lo. |
| **CPF**          | Value Object                 | Sempre pertence a apenas um paciente e não muda. |

---
## Exercicio

```csharp
public class Cliente
{
    public Guid Id { get; private set; }
    public string Nome { get; private set; }
    public Documento Documento { get; private set; }
    public DateTime DataNascimento { get; private set; }

    public Cliente(string nome, Documento documento, DateTime dataNascimento)
    {
        if (string.IsNullOrWhiteSpace(nome))
            throw new ArgumentNullException("Necessário preencher o nome.");

        if (dataNascimento == DateTime.MinValue || dataNascimento > DateTime.Now)
            throw new ArgumentNullException("Data de Nascimento inválido.");

        Id = Guid.NewGuid();
        Nome = nome;
        Documento = documento;
        DataNascimento = dataNascimento;
    }


}

public class Localizacao
{
    public string CEP { get; private set; }
    public string Rua { get; private set; }
    public string Bairro { get; private set; }
    public string Cidade { get; private set; }
    public string Estado { get; private set; }
    public string Numero { get; private set; }
    public string Complemento { get; private set; }

    public Localizacao(string cep, string rua, string bairro, string cidade, string estado, string numero, string complemento)
    {
        CEP = string.IsNullOrWhiteSpace(cep) ? throw new ArgumentNullException(nameof(cep)) : cep;
        Rua = string.IsNullOrWhiteSpace(rua) ? throw new ArgumentNullException(nameof(rua)) : rua;
        Bairro = string.IsNullOrWhiteSpace(bairro) ? throw new ArgumentNullException(nameof(bairro)) : bairro;
        Cidade = string.IsNullOrWhiteSpace(cidade) ? throw new ArgumentNullException(nameof(cidade)) : cidade;
        Estado = string.IsNullOrWhiteSpace(estado) ? throw new ArgumentNullException(nameof(estado)) : estado;
        Numero = string.IsNullOrWhiteSpace(numero) ? throw new ArgumentNullException(nameof(numero)) : numero;
        Complemento = string.IsNullOrWhiteSpace(complemento) ? throw new ArgumentNullException(nameof(complemento)) : complemento;
    }
}

public class Estabelecimento
{
    public Guid Id { get; private set; }
    public Localizacao Localizacao { get; private set; }
    public string RazaoSocial { get; private set; }
    public string Descricao { get; private set; }
    public Documento Documento { get; private set; }

    public Estabelecimento(Localizacao localizacao, string razaoSocial, string descricao, Documento documento)
    {
        Id = Guid.NewGuid();
        Localizacao = localizacao ?? throw new ArgumentNullException(nameof(localizacao));
        RazaoSocial = string.IsNullOrWhiteSpace(razaoSocial) ? throw new ArgumentNullException(nameof(razaoSocial)) : razaoSocial;
        Descricao = string.IsNullOrWhiteSpace(descricao) ? throw new ArgumentNullException(nameof(descricao)) : descricao;
        Documento = documento ?? throw new ArgumentNullException(nameof(documento));
    }
}

public class Produto
{
    public Guid Id { get; private set; }
    public string Nome { get; private set; }
    public string Descricao { get; private set; }
    public decimal Preco { get; private set; }
    public bool EhVegano { get; private set; }
    public bool EhSemGluten { get; private set; }
    public bool Disponivel { get; private set; } = true;

    public Produto(string nome, string descricao, decimal preco, bool ehVegano, bool ehSemGluten, bool disponivel)
    {
        if (preco <= 0)
            throw new ArgumentException("Preço deve ser maior que 0.");

        Id = Guid.NewGuid();
        Nome = string.IsNullOrWhiteSpace(nome) ? throw new ArgumentNullException(nameof(nome)) : nome;
        Descricao = string.IsNullOrWhiteSpace(descricao) ? throw new ArgumentNullException(nameof(descricao)) : descricao;
        Preco = preco;
        EhVegano = ehVegano;
        EhSemGluten = ehSemGluten;
        Disponivel = disponivel;
    }
    public void MarcarIndisponivel() => Disponivel = false;
    public void MarcarDisponivel() => Disponivel = true;
}

public class Pedido
{
    public Estabelecimento Estabelecimento { get; private set; }
    public Cliente Cliente { get; private set; }
    public List<Produto> Produtos { get; private set; }
    public decimal ValorTotal { get; private set; }
    public decimal Taxa { get; private set; }
    public DateTime DataPedido { get; private set; } = DateTime.Now;

    public Pedido(Estabelecimento estabelecimento, Cliente cliente, List<Produto> produtos, decimal valorTotal, decimal taxa)
    {
        Produtos = produtos ?? throw new ArgumentNullException(nameof(produtos));

        if (produtos.Count <= 0)
            throw new ArgumentException("Necessário ter ao menos um produto vinculado ao produto.");
        if (valorTotal <= 0)
            throw new ArgumentException("Necessário informar o valor total.");

        ValorTotal = valorTotal;
        Taxa = taxa;
    }

    public void AdicionarProduto(Produto produto){
        Produtos.Add(produto);
    }
    public void RemoverProdutos(Produto produto) {
        Produtos.Remove(produto);
    }
}

public class Documento
{
    public TipoDocumento Tipo { get; private set; }
    public string Numero { get; private set; }

    public Documento(TipoDocumento tipo, string numero)
    {

        if (string.IsNullOrWhiteSpace(numero))
            throw new ArgumentNullException("Necessário preencher o número.");

        if (tipo == TipoDocumento.CPF && !Regex.IsMatch(numero, "^[0-9]{11}$"))
            throw new ArgumentException("CPF inválido!");

        if (tipo == TipoDocumento.CNPJ && !Regex.IsMatch(numero, "^[0-9]{14}$"))
            throw new ArgumentException("CNPJ inválido!");

        Tipo = tipo;
        Numero = numero;
    }
}
public enum TipoDocumento
{
    CPF,
    CNPJ
}
```

| **Elemento**      | **Entidade ou Value Object?** | **Justificativa** |
|------------------|-----------------------------|-------------------|
| **Cliente**     | Entidade                     | Tem um ID único e pode mudar seus dados ao longo do tempo. |
| **Estabelecimento**       | Entidade                     | Possui identidade única e pode atualizar sua especialidade. |
| **Produto**       | Entidade                     | Possui identidade única e pode atualizar sua especialidade. |
| **Localização**          | Value Object | Sempre pertence a apenas um paciente e não muda. |
| **Documento**          | Value Object | Sempre pertence a apenas um paciente e não muda. |
| **Pedido**          | Aggregate Root                 | Sempre pertence a apenas um paciente e não muda. |
| **Reserva**     | Aggregate Root  | Se um paciente muda de endereço, não faz sentido modificar o antigo, apenas substituí-lo. |
| **Fila**          | Aggregate Root | Sempre pertence a apenas um paciente e não muda. |

---


### **🏗️ Agregados e Aggregate Root**
- Um **Agregado** agrupa entidades e objetos de valor que fazem parte de uma mesma regra de consistência.  
- O **Aggregate Root** é a entidade principal que controla a consistência do agregado.  

📌 **Exemplo de Agregado para o Contexto de Consultas Médicas:**  
- **Consulta** (Aggregate Root)  
  - Médico (Entidade)  
  - Paciente (Entidade)  
  - Data da Consulta (Value Object)  
  - Diagnóstico (Value Object)  

📌 **Regras de Negócio no Agregado:**  
- Um **médico** só pode ter **uma consulta ativa por horário**.  
- Uma **consulta finalizada** não pode ser alterada.  

---

### **🗃️ Repositórios**
Os **Repositórios** são responsáveis por **persistir e recuperar** agregados.  
✅ Devem **trabalhar apenas com Aggregate Roots**.  
✅ **Não devem expor entidades internas do agregado diretamente**.  

💡 **Exemplo de Interface de Repositório para o Contexto de Consultas:**  

```csharp
public interface IConsultaRepository
{
    Consulta ObterPorId(Guid id);
    void Salvar(Consulta consulta);
}
```

```csharp
public interface IReservaRepository
{
    Reserva ObterPorId(Guid id);
    List<Reserva> ObterPorEstabelecimento(Estabelecimento estabelecimento);
    void Reservar(Reserva reserva); 
}
```

📌 **Por que o Repositório trabalha apenas com `Consulta`?**  
- Porque **Consulta é o Aggregate Root**, então **Paciente e Médico são gerenciados por ele**.  

---

## **📝 2️⃣ Atividade Prática: Modelagem do Domínio**

📌 **Objetivo:**  
Criar as **Entidades, Value Objects, Agregados e Repositórios** do seu projeto.  

📌 **Instruções:**  
1️⃣ **Identifique as Entidades e Value Objects** do seu domínio.  
2️⃣ **Defina os Agregados e seu Aggregate Root**.  
3️⃣ **Implemente um diagrama mostrando as relações entre os elementos**.  
4️⃣ **Crie a interface do repositório para persistência do agregado**.  

📌 **Exemplo de Resposta para Keller’s Health:**  

| **Elemento**            | **Tipo**         | **Explicação** |
|-------------------------|-----------------|---------------|
| Paciente               | Entidade        | Possui identidade única e pode mudar ao longo do tempo. |
| Médico                 | Entidade        | Tem uma identidade única e pode alterar seus horários. |
| CPF                    | Value Object    | Não muda e sempre pertence a um único paciente. |
| Endereço               | Value Object    | Se o paciente mudar de endereço, um novo objeto será criado. |
| Consulta (Agregado)    | Aggregate Root  | Controla a relação entre Paciente, Médico e Data da Consulta. |


📌 **Ferramentas para Criar o Diagrama:**  
- [Miro](https://miro.com/)  
- [Lucidchart](https://www.lucidchart.com/)  
- [Figma](https://www.figma.com/)  
