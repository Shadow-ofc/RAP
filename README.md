# RAP
sistema de gerenciamento e leitura de agua

import java.util.*;
import java.time.LocalDate;
class BusinessException extends Exception {
    public BusinessException(String message) {
        super(message);
    }
}
enum StatusFatura {
    PENDENTE, PAGO, ATRASADO, CANCELADO
}
class Cliente {
    private String id;
    private String nome;
    private String cpf;
    private String endereco;
    private String numeroContador;
    public Cliente(String id, String nome, String cpf, String endereco, String numeroContador) {
        this.id = id;
        this.nome = nome;
        this.cpf = cpf;
        this.endereco = endereco;
        this.numeroContador = numeroContador;
    }
    public String getId() {
        return id;
    }
    public String getNome() {
        return nome;
    }
    @Override
    public String toString() {
        return "ID: " + id +
                "\nNome: " + nome +
                "\nCPF: " + cpf +
                "\nEndereço: " + endereco +
                "\nContador: " + numeroContador;
    }
}
class Fatura {
    private String id;
    private String idCliente;
    private double consumoM3;
    private double valorBase;
    private double impostos;
    private double multa;
    private StatusFatura status;
    private LocalDate dataVencimento;
    public Fatura(String id, String idCliente, double consumoM3, double tarifa) {
        this.id = id;
        this.idCliente = idCliente;
        this.consumoM3 = consumoM3;
        this.valorBase = consumoM3 * tarifa;
        this.impostos = this.valorBase * 0.14;
        this.multa = 0.0;
        this.status = StatusFatura.PENDENTE;
        this.dataVencimento = LocalDate.now().plusDays(15);
    }
    public String getId() {
        return id;
    }
    public String getIdCliente() {
        return idCliente;
    }
    public double getValorTotal() {
        return valorBase + impostos + multa;
    }
    public StatusFatura getStatus() {
        return status;
    }
    public void setStatus(StatusFatura status) {
        this.status = status;
    }
    public LocalDate getDataVencimento() {
        return dataVencimento;
    }
    public void aplicarMulta(double valor) {
        this.multa = valor;
    }
    @Override
    public String toString() {
        return "\nFATURA: " + id +
                "\nCliente: " + idCliente +
                "\nConsumo: " + consumoM3 + " m3" +
                "\nValor Total: " + getValorTotal() +
                "\nStatus: " + status +
                "\nVencimento: " + dataVencimento;
    }
}
class GestorAguaService {
    private Map<String, Cliente> repositorioClientes = new HashMap<>();
    private List<Fatura> repositorioFaturas = new ArrayList<>();
    private static final double TARIFA_PADRAO = 45.50;
    public void registrarCliente(Cliente cliente) throws BusinessException {
        if (repositorioClientes.containsKey(cliente.getId())) {
            throw new BusinessException("ID de cliente já existente.");
        }
        repositorioClientes.put(cliente.getId(), cliente);
    }
    public Fatura emitirFatura(String idCliente, double consumo) throws BusinessException {
        if (!repositorioClientes.containsKey(idCliente)) {
            throw new BusinessException("Cliente não localizado.");
        }
        String idFatura = "FAT-" +
                UUID.randomUUID().toString().substring(0, 5).toUpperCase();
        Fatura novaFatura = new Fatura(
                idFatura,
                idCliente,
                consumo,
                TARIFA_PADRAO
        );
        repositorioFaturas.add(novaFatura);
        return novaFatura;
    }
    public void processarPagamento(String idFatura, double valor)
            throws BusinessException {
        Fatura fatura = repositorioFaturas.stream()
                .filter(f -> f.getId().equals(idFatura))
                .findFirst()
                .orElseThrow(() ->
                        new BusinessException("Fatura não encontrada."));
        if (fatura.getStatus() == StatusFatura.PAGO) {
            throw new BusinessException("Fatura já paga.");
        }
        if (LocalDate.now().isAfter(fatura.getDataVencimento())) {
            fatura.aplicarMulta(fatura.getValorTotal() * 0.02);
            fatura.setStatus(StatusFatura.ATRASADO);
        }
        if (valor < fatura.getValorTotal()) {
            throw new BusinessException(
                    "Valor insuficiente. Total: "
                            + fatura.getValorTotal()
            );
        }
        fatura.setStatus(StatusFatura.PAGO);
    }
    public void listarClientes() {
        if (repositorioClientes.isEmpty()) {
            System.out.println("Nenhum cliente cadastrado.");
            return;
        }
        for (Cliente c : repositorioClientes.values()) {
            System.out.println("\n------------------");
            System.out.println(c);
        }
    }
    public void listarFaturas() {
        if (repositorioFaturas.isEmpty()) {
            System.out.println("Nenhuma fatura registrada.");
            return;
        }
        for (Fatura f : repositorioFaturas) {
            System.out.println("\n------------------");
            System.out.println(f);
        }
    }
    public void mostrarResumo() {
        System.out.println("\n===== RESUMO =====");
        System.out.println("Clientes cadastrados: "
                + repositorioClientes.size());
        System.out.println("Faturas registradas: "
                + repositorioFaturas.size());
        double receita = repositorioFaturas.stream()
                .filter(f -> f.getStatus() == StatusFatura.PAGO)
                .mapToDouble(Fatura::getValorTotal)
                .sum();
        System.out.println("Receita total: " + receita);
    }
}
public class SistemaGestaoAgua {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        GestorAguaService service = new GestorAguaService();
        int opcao;
        do {
            System.out.println("\n===== SISTEMA DE GESTÃO DE ÁGUA =====");
            System.out.println("1 - Registrar Cliente");
            System.out.println("2 - Emitir Fatura");
            System.out.println("3 - Processar Pagamento");
            System.out.println("4 - Listar Clientes");
            System.out.println("5 - Listar Faturas");
            System.out.println("6 - Mostrar Resumo");
            System.out.println("0 - Sair");
            System.out.print("Escolha: ");
            opcao = sc.nextInt();
            sc.nextLine();
            try {
                switch (opcao) {
                    case 1:
                        System.out.println("\n=== REGISTRO DE CLIENTE ===");
                        System.out.print("ID: ");
                        String id = sc.nextLine();
                        System.out.print("Nome: ");
                        String nome = sc.nextLine();
                        System.out.print("CPF: ");
                        String cpf = sc.nextLine();
                        System.out.print("Endereço: ");
                        String endereco = sc.nextLine();
                        System.out.print("Número do contador: ");
                        String contador = sc.nextLine();
                        Cliente cliente = new Cliente(
                                id,
                                nome,
                                cpf,
                                endereco,
                                contador
                        );
                        service.registrarCliente(cliente);
                        System.out.println("Cliente registrado com sucesso!");
                        break;
                    case 2:
                        System.out.println("\n=== EMISSÃO DE FATURA ===");
                        System.out.print("ID do cliente: ");
                        String idCliente = sc.nextLine();
                        System.out.print("Consumo em m3: ");
                        double consumo = sc.nextDouble();
                        sc.nextLine();
                        Fatura fatura = service.emitirFatura(
                                idCliente,
                                consumo
                        );
                        System.out.println("Fatura emitida com sucesso!");
                        System.out.println(fatura);
                        break;
                    case 3:
                        System.out.println("\n=== PROCESSAMENTO DE PAGAMENTO ===");
                        System.out.print("ID da fatura: ");
                        String idFatura = sc.nextLine();
                        System.out.print("Valor pago: ");
                        double valor = sc.nextDouble();
                        sc.nextLine();
                        service.processarPagamento(idFatura, valor);
                        System.out.println("Pagamento realizado com sucesso!");
                        break;
                    case 4:
                        System.out.println("\n=== LISTA DE CLIENTES ===");
                        service.listarClientes();
                        break;
                    case 5:
                        System.out.println("\n=== LISTA DE FATURAS ===");
                        service.listarFaturas();
                        break;
                    case 6:
                        service.mostrarResumo();
                        break;
                    case 0:
                        System.out.println("Sistema encerrado.");
                        break;
                    default:
                        System.out.println("Opção inválida.");
                }
            } catch (BusinessException e) {
                System.out.println("Erro: " + e.getMessage());
            } catch (Exception e) {
                System.out.println("Erro inesperado.");
            }
        } while (opcao != 0);
        sc.close();
}
}
