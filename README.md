- 👋 Hi, I’m @Blyscoin
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Blyscoin/Blyscoin is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

contract Blyscoin {
    string public name = "Blyscoin";
    string public symbol = "BLYS";
    uint8 public decimals = 9;
    uint256 public totalSupply;
    uint256 public constant MAX_SUPPLY = 1_000_000_000 * 10**9; // 1 bilh￣o de BLYS com 9 casas decimais
    uint256 public totalBurned = 0;
    uint256 public maxBurnRate = 2; // M￡ximo de 2% a ser queimado por transa￧￣o

    address public owner;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    mapping(address => bool) public excludedFromBurn; // Endere￧os exclu￭dos da queima

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event TokensBurned(uint256 amount);

    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }

    constructor() {
        owner = msg.sender;
        totalSupply = MAX_SUPPLY;
        balanceOf[msg.sender] = totalSupply;
    }

    // Fun￧￣o para transferir tokens
    function transfer(address recipient, uint256 amount) public returns (bool) {
        require(recipient != address(0), "Invalid address");
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");

        uint256 burnAmount = 0;
        
        // Se o endere￧o n￣o for exclu￭do da queima, aplique a taxa de queima
        if (!excludedFromBurn[msg.sender] && !excludedFromBurn[recipient]) {
            burnAmount = (amount * maxBurnRate) / 100;
        }

        uint256 amountAfterBurn = amount - burnAmount;

        // Se houver tokens a serem queimados, queim￡-los
        if (burnAmount > 0) {
            balanceOf[msg.sender] -= burnAmount;
            totalBurned += burnAmount;
            emit TokensBurned(burnAmount);
        }

        balanceOf[msg.sender] -= amountAfterBurn;
        balanceOf[recipient] += amountAfterBurn;
        emit Transfer(msg.sender, recipient, amountAfterBurn);

        return true;
    }

    // Fun￧￣o para aprovar outra conta a gastar tokens em nome do usu￡rio
    function approve(address spender, uint256 amount) public returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    // Fun￧￣o para transferir tokens de uma conta aprovada
    function transferFrom(address sender, address recipient, uint256 amount) public returns (bool) {
        require(sender != address(0), "Invalid address");
        require(recipient != address(0), "Invalid address");
        require(balanceOf[sender] >= amount, "Insufficient balance");
        require(allowance[sender][msg.sender] >= amount, "Allowance exceeded");

        uint256 burnAmount = 0;
        
        // Se o endere￧o n￣o for exclu￭do da queima, aplique a taxa de queima
        if (!excludedFromBurn[sender] && !excludedFromBurn[recipient]) {
            burnAmount = (amount * maxBurnRate) / 100;
        }

        uint256 amountAfterBurn = amount - burnAmount;

        // Se houver tokens a serem queimados, queim￡-los
        if (burnAmount > 0) {
            balanceOf[sender] -= burnAmount;
            totalBurned += burnAmount;
            emit TokensBurned(burnAmount);
        }

        balanceOf[sender] -= amountAfterBurn;
        balanceOf[recipient] += amountAfterBurn;
        allowance[sender][msg.sender] -= amount;
        emit Transfer(sender, recipient, amountAfterBurn);

        return true;
    }

    // Fun￧￣o para queimar tokens
    function burn(uint256 amount) external onlyOwner {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        balanceOf[msg.sender] -= amount;
        totalBurned += amount;
        emit TokensBurned(amount);
    }

    // Fun￧￣o para distribuir o airdrop
    function airdrop(address[] calldata recipients, uint256 amount) external onlyOwner {
        uint256 totalAmount = amount * recipients.length;
        require(totalAmount <= balanceOf[msg.sender], "Not enough balance for airdrop");

        for (uint256 i = 0; i < recipients.length; i++) {
            balanceOf[msg.sender] -= amount;
            balanceOf[recipients[i]] += amount;
            emit Transfer(msg.sender, recipients[i], amount);
        }
    }

    // Fun￧￣o para excluir endere￧os da queima
    function excludeFromBurn(address account, bool excluded) external onlyOwner {
        excludedFromBurn[account] = excluded;
    }

    // Fun￧￣o para ver o total de tokens queimados
    function getTotalBurned() external view returns (uint256) {
        return totalBurned;
    }

    // Fun￧￣o para bloquear qualquer tentativa de mintar tokens ap￳s o lan￧amento
    function mint(address account, uint256 amount) public pure {
        revert("Minting is not allowed after deployment.");
    }
}
