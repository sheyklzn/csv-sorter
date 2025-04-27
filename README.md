#include <stdio.h>
#include <stdlib.h>
#include <string.h> // Para a manipulação de strings

// Definição da estrutura para representar o produto
typedef struct
{
    int codigo;
    char nome[50];
    char descricao[100];
    float preco_custo;
    float preco_venda;
    int quantidade_estoque;
    char categoria[30];
    /* data */
} Produto;

#define MAX_PRODUTOS 100 // Define um limite máximo de produtos

Produto produtos[MAX_PRODUTOS];
int numero_produtos = 0; // Para controlar quantos produtos estão cadastrados

void ExibirmenuPrincipal(){
 printf("\n--- Menu Principal ---\n");
    printf("1. Adicionar Produto\n");
    printf("2. Listar Produtos\n");
    printf("3. Buscar Produto\n");
    printf("4. Atualizar Produto\n");
    printf("5. Remover Produto\n");
    printf("6. Iniciar Venda\n");
    printf("7. Listar Vendas\n");
    printf("8. Buscar Venda\n");
    printf("9. Sair\n");
    printf("Escolha uma opcao: ");
}

void AdicionarProduto() {
    if (numero_produtos < MAX_PRODUTOS) {
        Produto novoProduto;

        printf("\n--- Adicionar Novo Produto ---\n");

        printf("Codigo: ");
        if (scanf("%d", &novoProduto.codigo) != 1) {
            printf("Entrada invalida para o código.\n");
            // Lidar com o erro de entrada, talvez limpando o buffer e retornando
            while (getchar() != '\n');
            return;
        }
        getchar(); // Limpar o buffer do teclado

        printf("Nome: ");
        if (fgets(novoProduto.nome, sizeof(novoProduto.nome), stdin) != NULL) {
            novoProduto.nome[strcspn(novoProduto.nome, "\n")] = 0; // Remover a nova linha
        } else {
            printf("Erro ao ler o nome.\n");
            return;
        }

        printf("Descricao: ");
        if (fgets(novoProduto.descricao, sizeof(novoProduto.descricao), stdin) != NULL) {
            novoProduto.descricao[strcspn(novoProduto.descricao, "\n")] = 0;
        } else {
            printf("Erro ao ler a descricao.\n");
            return;
        }

        printf("Preco de Custo: ");
        if (scanf("%f", &novoProduto.preco_custo) != 1) {
            printf("Entrada invalida para o preço de custo.\n");
            while (getchar() != '\n');
            return;
        }

        printf("Preco de Venda: ");
        if (scanf("%f", &novoProduto.preco_venda) != 1) {
            printf("Entrada invalida para o preço de venda.\n");
            while (getchar() != '\n');
            return;
        }

        printf("Quantidade em Estoque: ");
        if (scanf("%d", &novoProduto.quantidade_estoque) != 1) {
            printf("Entrada invalida para a quantidade em estoque.\n");
            while (getchar() != '\n');
            return;
        }
        getchar(); // Limpar o buffer

        printf("Categoria: ");
        if (fgets(novoProduto.categoria, sizeof(novoProduto.categoria), stdin) != NULL) {
            novoProduto.categoria[strcspn(novoProduto.categoria, "\n")] = 0;
        } else {
            printf("Erro ao ler a categoria.\n");
            return;
        }

        produtos[numero_produtos] = novoProduto;
        numero_produtos++;

        printf("Produto adicionado com sucesso!\n");
    } else {
        printf("Limite maximo de produtos atingido!\n");
    }
}
void ListarProdutos() {
    printf("\n--- Listagem de Produtos ---\n");
    if (numero_produtos > 0) {
        printf("%-10s %-50s %-10s %-10s %-10s \n",
               "Codigo", "Nome", "Preco Venda", "Estoque" , "Categoria");
        for (int i = 0; i < numero_produtos; i++) {
            printf("%-10d %-50s %-10.2f %-10d %-10s\n",
                   produtos[i].codigo, produtos[i].nome, produtos[i].preco_venda,
                   produtos[i].quantidade_estoque, produtos[i].categoria);
        }
    } else {
        printf("Nenhum produto cadastrado.\n");
    }
}

void BuscarProduto() {
    int codigoBusca;
    printf("\n--- Buscar Produto ---\n");
    printf("Digite o codigo do produto que deseja buscar: ");
    if (scanf("%d", &codigoBusca) == 1) {
        int encontrado = 0;
        for (int i = 0; i < numero_produtos; i++) {
            if (produtos[i].codigo == codigoBusca) {
                printf("\n--- Produto Encontrado ---\n");
                printf("Codigo: %d\n", produtos[i].codigo);
                printf("Nome: %s\n", produtos[i].nome);
                printf("Descricao: %s\n", produtos[i].descricao);
                printf("Preco de Custo: %.2f\n", produtos[i].preco_custo);
                printf("Preco de Venda: %.2f\n", produtos[i].preco_venda);
                printf("Quantidade em Estoque: %d\n", produtos[i].quantidade_estoque);
                printf("Categoria: %s\n", produtos[i].categoria);
                encontrado = 1;
                break;
            }
        }
        if (!encontrado) {
            printf("Produto com o codigo %d nao encontrado.\n", codigoBusca);
        }
    } else {
        printf("Codigo de busca invalido.\n");
        while (getchar() != '\n'); // Limpar o buffer
    }
    getchar(); // Limpar o buffer
}

void AtualizarProduto() {
    int codigoAtualizar;
    printf("\n--- Atualizar Produto ---\n");
    printf("Digite o codigo do produto que deseja atualizar: ");
    if (scanf("%d", &codigoAtualizar) == 1) {
        int encontrado = 0;
        for (int i = 0; i < numero_produtos; i++) {
            if (produtos[i].codigo == codigoAtualizar) {
                printf("\n--- Editar Produto (Codigo: %d) ---\n", codigoAtualizar);
                // Permitir que o usuário atualize os campos desejados
                printf("Novo nome (deixe em branco para manter): ");
                getchar(); // Limpar o buffer
                char novoNome[50];
                fgets(novoNome, sizeof(novoNome), stdin);
                novoNome[strcspn(novoNome, "\n")] = 0;
                if (strlen(novoNome) > 0) {
                    strcpy(produtos[i].nome, novoNome);
                }

                printf("Nova descricao (deixe em branco para manter): ");
                char novaDescricao[100];
                fgets(novaDescricao, sizeof(novaDescricao), stdin);
                novaDescricao[strcspn(novaDescricao, "\n")] = 0;
                if (strlen(novaDescricao) > 0) {
                    strcpy(produtos[i].descricao, novaDescricao);
                }

                printf("Novo preco de custo (digite 2 para manter): ");
                float novoPrecoCusto;
                if (scanf("%f", &novoPrecoCusto) == 1 && novoPrecoCusto != 2) {
                    produtos[i].preco_custo = novoPrecoCusto;
                }

                printf("Novo preco de venda (digite 2 para manter): ");
                float novoPrecoVenda;
                if (scanf("%f", &novoPrecoVenda) == 1 && novoPrecoVenda != 2) {
                    produtos[i].preco_venda = novoPrecoVenda;
                }

                printf("Nova quantidade em estoque (digite -1 para manter): ");
                int novaQuantidadeEstoque;
                if (scanf("%d", &novaQuantidadeEstoque) == 1 && novaQuantidadeEstoque != -1) {
                    produtos[i].quantidade_estoque = novaQuantidadeEstoque;
                }
                getchar(); // Limpar o buffer

                printf("Nova categoria (deixe em branco para manter): ");
                char novaCategoria[30];
                fgets(novaCategoria, sizeof(novaCategoria), stdin);
                novaCategoria[strcspn(novaCategoria, "\n")] = 0;
                if (strlen(novaCategoria) > 0) {
                    strcpy(produtos[i].categoria, novaCategoria);
                }

                printf("Produto atualizado com sucesso!\n");
                encontrado = 1;
                break;
            }
        }
        if (!encontrado) {
            printf("Produto com o codigo %d nao encontrado.\n", codigoAtualizar);
        }
    } else {
        printf("Codigo de atualizacao invalido.\n");
        while (getchar() != '\n'); // Limpar o buffer
    }
    getchar(); // Limpar o buffer
}

void RemoverProduto() {
    int codigoRemover;
    printf("\n--- Remover Produto ---\n");
    printf("Digite o codigo do produto que deseja remover: ");
    if (scanf("%d", &codigoRemover) == 1) {
        int encontrado = 0;
        for (int i = 0; i < numero_produtos; i++) {
            if (produtos[i].codigo == codigoRemover) {
                // Deslocar os produtos subsequentes para preencher o espaço
                for (int j = i; j < numero_produtos - 1; j++) {
                    produtos[j] = produtos[j + 1];
                }
                numero_produtos--;
                printf("Produto com o codigo %d removido com sucesso!\n", codigoRemover);
                encontrado = 1;
                break;
            }
        }
        if (!encontrado) {
            printf("Produto com o codigo %d nao encontrado.\n", codigoRemover);
        }
    } else {
        printf("Codigo de remocao invalido.\n");
        while (getchar() != '\n'); // Limpar o buffer
    }
    getchar(); // Limpar o buffer
}

// --- Funções relacionadas a Vendas (estrutura Venda não definida ainda) ---

void IniciarVenda() {
    printf("\n--- Iniciar Venda ---\n");
    printf("Funcionalidade Iniciar Venda ainda nao completamente implementada.\n");
    // Aqui você implementaria a lógica para iniciar uma nova venda,
    // selecionar produtos, quantidades, calcular total, etc.
}

void ListarVendas() {
    printf("\n--- Listagem de Vendas ---\n");
    printf("Funcionalidade Listar Vendas ainda nao implementada.\n");
    // Aqui você implementaria a lógica para listar as vendas registradas.
}

void BuscarVendas() {
    printf("\n--- Buscar Venda ---\n");
    printf("Funcionalidade Buscar Venda ainda nao implementada.\n");
    // Aqui você implementaria a lógica para buscar vendas por código, data, etc.
}

int main() {
	
 int opcao;

    printf("Sistema de Cadastro de Produtos e Vendas\n");
    
    do {
        ExibirmenuPrincipal();
        if (scanf("%d", &opcao) != 1) {
            printf("Opcaoo invalida. Por favor, digite um numero.\n");
            // Limpar o buffer do teclado
            while (getchar() != '\n');
            opcao = 0; // Para continuar o loop
            continue;
        }
        getchar(); // Limpar o buffer

        switch (opcao) {
            case 1:
                AdicionarProduto();
                break;
            case 2:
                // Implementar a função listarProdutos
                ListarProdutos();
                break;
            case 3:
                // Implementar a função buscarProduto
                BuscarProduto();
                break;
            case 4:
                // Implementar a função atualizarProduto
                AtualizarProduto();
                break;
            case 5:
                // Implementar a função removerProduto
                RemoverProduto();
                break;
            case 6:
                // Implementar a função iniciarVenda
                IniciarVenda();
                break;
            case 7:
                // Implementar a função listarVendas
                ListarVendas();
                break;
            case 8:
                // Implementar a função buscarVenda
                BuscarVendas();
                break;
            case 9:
                printf("Saindo do sistema...\n");
                break;
            default:
                printf("Opcao invalida. Por favor, escolha uma opçao do menu.\n");
        }
    } while (opcao != 9);

    return 0;

    AdicionarProduto(); // Chamando a função para adicionar um produto

    return 0;
}
