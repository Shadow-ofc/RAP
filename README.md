# RAP
sistema de gerenciamento e leitura de agua


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
