# SOLID 101 no Swift (desenvolvimento)
Este é um projeto para mostrar exemplos práticos das idéias do Uncle Bob na linguagem Swift 3.
 
O SOLID é um acrônimo criado pelo Uncle Bob(Robert C. Martin) para unir alguns padrões muito utilizados em design de software no paradigma Orientado à Objetos que visam facilitar o seu teste, manutenção e sua legibilidade do código. 

Este artigo tem como foco mostrar como são essas 5 ideias e como elas podem ser aplicadas na linguagem que tanto amamos, o Swift❤️.

Os conceitos desenvolvidos são: 
 - S - Single Responsability (Responsabilidade Única)
 - O - Open/Closed = (Aberto e fechado)
 - L - Liskov Substitution (Substituição de Liskov)
 - I - Interface Segregation (Segregação de interface)
 - D - Dependecy Inversion (Inversão de dependência)

Estes princípios foram feitos para tentar acabar com alguns problemas da má arquitetura como:

 - Fragilidade: Uma mudança no código pode acabar quebrando várias
   partes do código.
 - Código Engessado: Que é um código escrito que não é possível de ser reaproveitado.
 - Rigidez: Uma mudança requer um enorme esforço pois ela acaba afetando diversas partes do código

O próprio Uncle Bob, não acredita que isso são regras, já que cada caso é um caso,  apenas acredita que estas ideias podem melhorar sua arquitetura e com isso tornar o ato de desenvolvimento e manutenção muito mais agradável para o desenvolvedor. 

**S**ingle Responsability ou Responsabilidade única
-----------------------------------------------
Uma classe deve ter apenas um, e somente um, motivo para mudar.
    - É a capacidade de cada classe ter apenas uma responsabilidade. Para isso, toda vez que uma classe for criada é necessário se perguntar: Quantas responsabilidades essa classe tem?

Vamos ver uma classe pronta: 

    class Handler {
        func handle() {
            let data = requestDataToAPI()
            let array = parse(data: data)
            self.saveToDB(array: array)
        }
        
        private func requestDataToAPI() -> Data {
            // Função faz a requisição para a API e retorna um Data
        }
        
        private func parse(data: Data) -> [String] {
            // Converte um Data para um array de Strings
        }
        
        private func saveToDB(array: [String]) {
            // Salva o array de strings no banco de dados (Realm/CoreData)
        }
    }

**Vamos nos perguntar? Quantas responsabilidades essa classe tem?**

 1. Pega os dados da API
 2. Parseia os dados da API
 3. Salva os dados no banco de dados
 
Isso acaba inferindo a idéia de responsabilidade única, pois em muitas vezes precisamos que esse código seja replicado, uma vez que raramente precisaremos fazer apenas uma vez uma requisição de dados.

Aplicando mais ao mundo real, podemos até considerar que usará o Alamofire para fazer a requisição, ObjectMapper pra fazer o parse e salva em um banco de dados em Realm ou CoreData.

Podemos resolver o problema, dividindo em diferentes classes, como no exemplo a seguir:

    class Handler {
        let apiHandler: APIHandler
        let parseHandler: ParseHandler
        let dbHandler: DBHandler
        
        init(apiHandler: APIHandler, parseHandler: ParseHandler, dbHandler: DBHandler) {
            self.apiHandler = apiHandler
            self.parseHandler = parseHandler
            self.dbHandler = dbHandler
        }
    }
    
    class APIHandler {
        func requestDataToAPI() -> Data {}
    }
    
    class ParseHandler {
        func parse(data: Data) -> [String] {}
    }
    
    class DBHandler {
        func saveToDB(array: [String]) {}
    }

**O que mudou?** 

Antes não era possível testar diretamente requestDataToAPI,  parse, saveToDB diretamente, pois eram métodos privados. Já com o código com responsabilidade única é possível testar todos os APIHandler, ParseHandler,  DBHandler diretamente e colocar por injeção de dependência para testar a classe Handler.


**O**pen/Closed - O principio do Aberto e fechado
---------------------------------------------
O que Uncle Bob quer dizer com aberto e fechado é: Aberto para para criar extensões e fechado para modificações.

Quando se pensa em um código fácil de fazer a manutenção é necessário pensar nessas duas características citadas anteriormente: 
 - Aberto para criar extensões: A classe deve estar aberta para extender
   comportamentos da classe sem esforços.
 - Fechado para modificações: Você deve estender a classe sem criar
   modificações.

Você pode conseguir essas características no seu código através de abstrações. 🙏🏿

Podemos ver nesse exemplo a criação de um cardápio de restaurante, a função imprimir cardápio está listando todas as descrições de cada pedido possível, como o hambúrguer.


    class Cardapio {
        func imprimeCardapio() {
            let hambugueres: [Hamburger] = {
                Hamburger(nome: "Big belly Burguer", molho: "Cisco", qtdHambugueres: 2),
                Hamburger(nome: "Breaking Burguer", molho: "Pinkman", qtdHambugueres: 2),
                Hamburger(nome: "How I Met Your Burguer", molho: "Marshall", qtdHambugueres: 2)
            }
            
            hambugueres.forEach { (hamburguer) in
                print(hamburguer.descricaoCardapio())
            }
        }
    }
    
    
    class Hamburger {
        let nome: String
        let molho: String
        let qtdHumbugueres: Int
        
        init(nome: String, molho: String, qtdHambugueres: Int) {
            self.nome = nome
            self.molho = molho
            self.qtdHumbugueres = qtdHumbugueres
        }
        
        
        func descricaoCardapio() -> String {
            return "Este prato tem o nome de: \(nome), tem um molho especial de \(molho) em cada um de seus deliciosos \(qtdHumbugueres). Experimente!"
        }
    }

Se você precisar criar um novo cardápio seria necessário alterar o código do descrição cardápio novamente e alterar a iteração que a imprime na classe  Cardápio. Isso vai acontecer toda vez que você criar uma nova classe. Como mostro no exemplo abaixo após adicionar uma classe PratoFeito:

    class Cardapio {
        func imprimeCardapio() {
            let hambugueres: [Hamburger] = {
                Hamburger(nome: "Big belly Burguer", molho: "Cisco", qtdHambugueres: 2),
                Hamburger(nome: "Breaking Burguer", molho: "Pinkman", qtdHambugueres: 2),
                Hamburger(nome: "How I Met Your Burguer", molho: "Marshall", qtdHambugueres: 2)
            }
            
            hambugueres.forEach { (hamburguer) in
                print(hamburguer.descricaoCardapio())
            }
            
            let pratosFeitos: [PratoFeito] = {
                PratoFeito(nome: "Bife à milanesa", acompanhamento: "Arroz"),
                PratoFeito(nome: "Calabresa", acompanhamento: "Arroz"),
                PratoFeito(nome: "Bife a Parmediana", acompanhamento: "Arroz")
            }
            
            pratosFeitos.forEach { (pf) in
                print(pf.descricaoCardapio())
            }
            
        }
    }
    
    
    class Hamburger {
        let nome: String
        let molho: String
        let qtdHumbugueres: Int
        
        init(nome: String, molho: String, qtdHambugueres: Int) {
            self.nome = nome
            self.molho = molho
            self.qtdHumbugueres = qtdHumbugueres
        }
        
        
        func descricaoCardapio() -> String {
            return "Este prato tem o nome de: \(nome), tem um molho especial de \(molho) em cada um de seus deliciosos \(qtdHumbugueres). Experimente!"
        }
    }
    
    class PratoFeito {
        let nome: String
        let acompanhamento: String
        
        init(nome: String, acompanhamento: String) {
            self.nome = nome
            self.acompanhamento = acompanhamento
        }
        
        func descricaoCardapio() -> String {
            return "Este prato feito tem o nome de \(nome) e pode vir com o acompanhamento \(acompanhamento). Experimente!"
        }
    }

Podemos resolver esse problema de alteração de classe através de uma abstração utilizando protocolos 💙. 

Pode-se criar um protocolo chamado Descritivel que terá uma função descriçãoCardápio para todos os itens que serão imprimiveis no cardápio.


    protocol Descritivel {
        func descricaoCardapio()
    }
    
    class Cardapio {
        func imprimeCardapio() {
            let descritiveis: [Descritivel] = {
                Hamburger(nome: "Big belly Burguer", molho: "Cisco", qtdHambugueres: 2),
                Hamburger(nome: "Breaking Burguer", molho: "Pinkman", qtdHambugueres: 2),
                Hamburger(nome: "How I Met Your Burguer", molho: "Marshall", qtdHambugueres: 2),
                PratoFeito(nome: "Bife à milanesa", acompanhamento: "Arroz"),
                PratoFeito(nome: "Calabresa", acompanhamento: "Arroz"),
                PratoFeito(nome: "Bife a Parmediana", acompanhamento: "Arroz")
            }
            
            descritiveis.forEach { (pf) in
                print(pf.descricaoCardapio())
            }
            
        }
    }
    
    
    class Hamburger: Descritivel {
        let nome: String
        let molho: String
        let qtdHumbugueres: Int
        
        init(nome: String, molho: String, qtdHambugueres: Int) {
            self.nome = nome
            self.molho = molho
            self.qtdHumbugueres = qtdHumbugueres
        }
        
        
        func descricaoCardapio() -> String {
            return "Este prato tem o nome de: \(nome), tem um molho especial de \(molho) em cada um de seus deliciosos \(qtdHumbugueres). Experimente!"
        }
    }
    
    class PratoFeito: Descritivel {
        let nome: String
        let acompanhamento: String
        
        init(nome: String, acompanhamento: String) {
            self.nome = nome
            self.acompanhamento = acompanhamento
        }
        
        func descricaoCardapio() -> String {
            return "Este prato feito tem o nome de \(nome) e pode vir com o acompanhamento \(acompanhamento). Experimente!"
        }
    }

Assim, você pode adicionar a quantidade de itens de classes diferentes que quiser, desde que sejam Descritiveis, para que consigam ser impressas no cardápio.

** Continua, estou desenvolvendo o resto **
