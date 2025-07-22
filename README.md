// SPDX-License-Identifier: MIT
pragma solidity ^0.8.21;

contract BancoMacroArgentina {
    struct Cliente {
        string nombre;
        uint256 dni;
        uint256 balance;
        uint256 reputacion;
        bool activo;
    }

    address public owner;
    mapping(address => Cliente) public clientes;
    uint256 public tasaInteres = 7; // ejemplo, anual
    uint256 public reputacionMinimaParaCredito = 600;

    modifier soloOwner() {
        require(msg.sender == owner, "Acceso denegado");
        _;
    }

    modifier clienteActivo(address cliente) {
        require(clientes[cliente].activo, "Cliente no activo");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function registrarCliente(address cuenta, string memory nombre, uint256 dni) public soloOwner {
        clientes[cuenta] = Cliente(nombre, dni, 0, 500, true);
    }

    function depositar(address cuenta) public payable clienteActivo(cuenta) {
        clientes[cuenta].balance += msg.value;
    }

    function solicitarCredito(address cuenta, uint256 monto) public clienteActivo(cuenta) {
        require(clientes[cuenta].reputacion >= reputacionMinimaParaCredito, "Reputacion insuficiente");
        clientes[cuenta].balance += monto;
    }

    function pagar(address cuenta, uint256 monto) public clienteActivo(cuenta) {
        require(clientes[cuenta].balance >= monto, "Fondos insuficientes");
        clientes[cuenta].balance -= monto;
    }

    function actualizarReputacion(address cuenta, uint256 nuevaRep) public soloOwner {
        clientes[cuenta].reputacion = nuevaRep;
    }

    function suspenderCuenta(address cuenta) public soloOwner {
        clientes[cuenta].activo = false;
    }

    function verDatosCliente(address cuenta) public view returns (
        string memory nombre,
        uint256 dni,
        uint256 balance,
        uint256 reputacion,
        bool activo
    ) {
        Cliente memory c = clientes[cuenta];
        return (c.nombre, c.dni, c.balance, c.reputacion, c.activo);
    }
}

