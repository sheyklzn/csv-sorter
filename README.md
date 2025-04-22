#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_LINHA 1024
#define MAX_COLUNAS 20

typedef struct {
    char **campos;
    int num_campos;
} Item;

typedef struct {
    Item *itens;
    int num_itens;
    int num_colunas;
} DadosCSV;

void liberar_dados(DadosCSV *dados) {
    for (int i = 0; i < dados->num_itens; i++) {
        for (int j = 0; j < dados->itens[i].num_campos; j++) {
            free(dados->itens[i].campos[j]);
        }
        free(dados->itens[i].campos);
    }
    free(dados->itens);
    free(dados);
}

DadosCSV *ler_csv(const char *nome_arquivo) {
    FILE *arquivo = fopen(nome_arquivo, "r");
    if (!arquivo) {
        perror("Erro ao abrir arquivo");
        return NULL;
    }

    DadosCSV *dados = malloc(sizeof(DadosCSV));
    if (!dados) {
        fclose(arquivo);
        return NULL;
    }

    dados->itens = NULL;
    dados->num_itens = 0;
    dados->num_colunas = 0;

    char linha[MAX_LINHA];
    while (fgets(linha, MAX_LINHA, arquivo)) {
        // Remove o \n do final
        linha[strcspn(linha, "\n")] = '\0';
        
        // Aloca espaço para o novo item
        dados->itens = realloc(dados->itens, (dados->num_itens + 1) * sizeof(Item));
        if (!dados->itens) {
            liberar_dados(dados);
            fclose(arquivo);
            return NULL;
        }

        Item *item = &dados->itens[dados->num_itens];
        item->campos = NULL;
        item->num_campos = 0;

        char *token;
        char *resto = linha;
        int dentro_aspas = 0;
        char *campo_atual = NULL;
        size_t tamanho_campo = 0;

        while (*resto) {
            if (*resto == '"') {
                dentro_aspas = !dentro_aspas;
                resto++;
                continue;
            }

            if (*resto == ',' && !dentro_aspas) {
                // Final do campo - adiciona ao item
                item->campos = realloc(item->campos, (item->num_campos + 1) * sizeof(char*));
                item->campos[item->num_campos] = campo_atual;
                item->num_campos++;
                campo_atual = NULL;
                tamanho_campo = 0;
                resto++;
            } else {
                // Adiciona caractere ao campo atual
                tamanho_campo++;
                campo_atual = realloc(campo_atual, tamanho_campo + 1);
                campo_atual[tamanho_campo - 1] = *resto;
                campo_atual[tamanho_campo] = '\0';
                resto++;
            }
        }

        // Adiciona o último campo
        if (campo_atual || item->num_campos == 0) {
            item->campos = realloc(item->campos, (item->num_campos + 1) * sizeof(char*));
            item->campos[item->num_campos] = campo_atual;
            item->num_campos++;
        }

        // Atualiza o número máximo de colunas
        if (item->num_campos > dados->num_colunas) {
            dados->num_colunas = item->num_campos;
        }

        dados->num_itens++;
    }

    fclose(arquivo);
    return dados;
}

int eh_numero(const char *str) {
    if (!str || !*str) return 0;
    char *endptr;
    strtod(str, &endptr);
    return *endptr == '\0';
}

int comparar_itens(const Item *a, const Item *b, int coluna, int crescente) {
    if (coluna >= a->num_campos || coluna >= b->num_campos) return 0;
    
    const char *campo_a = a->campos[coluna] ? a->campos[coluna] : "";
    const char *campo_b = b->campos[coluna] ? b->campos[coluna] : "";

    if (eh_numero(campo_a) && eh_numero(campo_b)) {
        double num_a = atof(campo_a);
        double num_b = atof(campo_b);
        return crescente ? (num_a > num_b) - (num_a < num_b) : (num_b > num_a) - (num_b < num_a);
    } else {
        return crescente ? strcmp(campo_a, campo_b) : strcmp(campo_b, campo_a);
    }
}

void ordenar_dados(DadosCSV *dados, int coluna, int crescente) {
    if (!dados || coluna < 0 || coluna >= dados->num_colunas) return;

    for (int i = 0; i < dados->num_itens - 1; i++) {
        for (int j = 0; j < dados->num_itens - i - 1; j++) {
            if (comparar_itens(&dados->itens[j], &dados->itens[j+1], coluna, crescente) > 0) {
                Item temp = dados->itens[j];
                dados->itens[j] = dados->itens[j+1];
                dados->itens[j+1] = temp;
            }
        }
    }
}

int gravar_csv(const char *nome_arquivo, const DadosCSV *dados) {
    FILE *arquivo = fopen(nome_arquivo, "w");
    if (!arquivo) {
        perror("Erro ao criar arquivo");
        return 0;
    }

    for (int i = 0; i < dados->num_itens; i++) {
        for (int j = 0; j < dados->itens[i].num_campos; j++) {
            if (j > 0) fprintf(arquivo, ",");
            if (dados->itens[i].campos[j]) {
                // Verifica se o campo contém vírgulas e, se sim, coloca entre aspas
                if (strchr(dados->itens[i].campos[j], ',')) {
                    fprintf(arquivo, "\"%s\"", dados->itens[i].campos[j]);
                } else {
                    fprintf(arquivo, "%s", dados->itens[i].campos[j]);
                }
            }
        }
        fprintf(arquivo, "\n");
    }

    fclose(arquivo);
    return 1;
}

void mostrar_menu(const DadosCSV *dados) {
    printf("\n=== Menu de Ordenação ===\n");
    printf("Colunas disponíveis:\n");
    for (int i = 0; i < dados->num_colunas; i++) {
        printf("%d. %s\n", i+1, dados->itens[0].campos[i] ? dados->itens[0].campos[i] : "(sem nome)");
    }
    printf("\n");
}

int main() {
    char nome_entrada[256];
    char nome_saida[256];
    int coluna, ordem;

    printf("Digite o nome do arquivo CSV de entrada: ");
    scanf("%255s", nome_entrada);

    DadosCSV *dados = ler_csv(nome_entrada);
    if (!dados) {
        printf("Erro ao ler o arquivo.\n");
        return 1;
    }

    if (dados->num_itens == 0) {
        printf("Arquivo vazio ou formato inválido.\n");
        liberar_dados(dados);
        return 1;
    }

    mostrar_menu(dados);

    printf("Digite o número da coluna para ordenar (1-%d): ", dados->num_colunas);
    scanf("%d", &coluna);
    coluna--; // Ajusta para índice 0-based

    if (coluna < 0 || coluna >= dados->num_colunas) {
        printf("Coluna inválida.\n");
        liberar_dados(dados);
        return 1;
    }

    printf("Ordenar em ordem:\n1. Crescente\n2. Decrescente\nEscolha: ");
    scanf("%d", &ordem);

    ordenar_dados(dados, coluna, ordem == 1);

    printf("Digite o nome do arquivo CSV de saída: ");
    scanf("%255s", nome_saida);

    if (gravar_csv(nome_saida, dados)) {
        printf("Arquivo gerado com sucesso: %s\n", nome_saida);
    } else {
        printf("Erro ao gerar arquivo.\n");
    }

    liberar_dados(dados);
    return 0;
}
