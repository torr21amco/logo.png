pragma solidity ^0.8.2;

contract AstroMineroToken {
    mapping(address => uint) public balances;
    mapping(address => mapping(address => uint)) public allowance;
    uint public totalSupply = 21000000 * 10 ** 18; // 21,000,000 tokens con 18 decimales
    string public name = "Astro Mining Coin";
    string public symbol = "AMCO";
    uint public decimals = 18;

    address public owner;
    address public liquidityAddress;
    uint public feePercentage = 3; // 0.3%

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    constructor(address _liquidityAddress) {
        owner = msg.sender;
        liquidityAddress = _liquidityAddress;
        balances[owner] = totalSupply;
    }
    
    function balanceOf(address owner) public view returns(uint) {
        return balances[owner];
    }

    function transfer(address to, uint value) public returns(bool) {
        uint fee = (value * feePercentage) / 1000;
        uint transferValue = value - fee;
        
        require(balanceOf(msg.sender) >= value, 'balance too low');
        
        balances[msg.sender] -= value;
        balances[to] += transferValue;
        balances[owner] += fee / 3;
        distributeFee(fee / 3);
        balances[liquidityAddress] += fee / 3;

        emit Transfer(msg.sender, to, transferValue);
        emit Transfer(msg.sender, owner, fee / 3);
        emit Transfer(msg.sender, address(this), fee / 3);
        emit Transfer(msg.sender, liquidityAddress, fee / 3);

        return true;
    }

    function transferFrom(address from, address to, uint value) public returns(bool) {
        uint fee = (value * feePercentage) / 1000;
        uint transferValue = value - fee;

        require(balanceOf(from) >= value, 'balance too low');
        require(allowance[from][msg.sender] >= value, 'allowance too low');

        balances[from] -= value;
        balances[to] += transferValue;
        balances[owner] += fee / 3;
        distributeFee(fee / 3);
        balances[liquidityAddress] += fee / 3;

        allowance[from][msg.sender] -= value;

        emit Transfer(from, to, transferValue);
        emit Transfer(from, owner, fee / 3);
        emit Transfer(from, address(this), fee / 3);
        emit Transfer(from, liquidityAddress, fee / 3);

        return true;   
    }

    function approve(address spender, uint value) public returns (bool) {
        allowance[msg.sender][spender] = value;
        emit Approval(msg.sender, spender, value);
        return true;   
    }

    function distributeFee(uint fee) internal {
        uint totalSupplyMinusOwner = totalSupply - balances[owner];
        for (uint i = 0; i < totalSupplyMinusOwner; i++) {
            // Distribute the fee to all token holders proportionally
            address holder = address(uint160(i));
            if (holder != owner && balances[holder] > 0) {
                uint holderShare = (fee * balances[holder]) / totalSupplyMinusOwner;
                balances[holder] += holderShare;
                emit Transfer(address(this), holder, holderShare);
            }
        }
    }
}

