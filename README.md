// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@chainlink/contracts/src/v0.8/shared/interfaces/LinkTokenInterface.sol"; 
import "@chainlink/contracts/src/v0.8/ChainlinkClient.sol";

contract InsurancePool is ERC20, Ownable {
    error InvalidAmount();
    error FailedSistem();
    error InvalidToken();
    error InvalidAddress();

    event TokensInvestorPurchased(address indexed buyer, uint256 Spent, uint256 tokensReceived);
    event CurrencyInvestorPurchased(address indexed buyer, uint256 currencyReceived, uint256 tokensSpent);

    constructor(
        string memory _name,
        string memory _symbol,
        address _initialOwner,
        address _token
    ) ERC20(_name, _symbol) Ownable(_initialOwner) {
        token = IERC20(_token);
    }

    IERC20 public immutable token;
    address public insuranceCore;

    function setInsuranceCore(address _core) external onlyOwner {
        require(_core != address(0), InvalidAddress());
        insuranceCore = _core;
    }

    function payOutInsurance(address passenger, uint256 amount) external {
        if (msg.sender != insuranceCore) revert FailedSistem();
        if (amount > token.balanceOf(address(this))) revert FailedSistem();

        require(token.transfer(passenger, amount), FailedSistem());
    }

    function Formula(uint256 amount) internal view returns (uint256) {
        if (amount == 0) revert InvalidAmount();

        uint256 totalLSupply = totalSupply();
        uint256 lpToMint;

        if (totalLSupply == 0) {
            lpToMint = amount;
        } else {
            uint256 currentPoolBalance = token.balanceOf(address(this));
            lpToMint = (amount * totalLpSupply) / currentPoolBalance;
        }

        return lpToMint;
    }

    function FormulaTo(uint256 amount) internal view returns(uint256) {
        if (amount == 0) revert InvalidAmount();

        uint256 totalLSupply = totalSupply();
        uint256 currentPoolBalance = token.balanceOf(address(this));
        uint256 amountToReturn;

        if (amount >= totalLSupply) revert InvalidToken();

        amountToReturn = (amount * currentPoolBalance) / totalLSupply;

        return amountToReturn;
    }

    function provideLiquidity(uint256 amount) public returns(uint256) {
        if (amount == 0) revert InvalidAmount();

        uint256 lpToMint = Formula(amount);

        bool success = token.transferFrom(msg.sender, address(this), amount);
        if (!success) revert FailedSistem();

        _mint(msg.sender, lpToMint);

        emit TokensInvestorPurchased(msg.sender, amount, lpToMint);

        return lpToMint;
    }

    function withdrawLiquidity(uint256 lpAmount) public returns(uint256) {
        if (lpAmount == 0) revert InvalidAmount();

        uint256 totalLSupply = totalSupply();
        uint256 amountToReturn = FormulaTo(lpAmount);

        if (lpAmount > balanceOf(msg.sender)) revert InvalidToken();

        _burn(msg.sender, lpAmount);

        bool success = token.transfer(msg.sender, amountToReturn);
        if (!success) revert FailedSistem();

        emit CurrencyInvestorPurchased(msg.sender, amountToReturn, lpAmount);

        return amountToReturn;
    }
}

contract FlightInsuranceCore is ChainlinkClient {
    using Chainlink for Chainlink.Request;

    IERC20 public immutable token;
    InsurancePool public immutable insurancePool;

    address private oracle;
    bytes32 private jobId;
    uint256 private fee;

    constructor(address _token, address _pool, address _chainlinkToken, address _oracle, string memory _jobId) {
        token = IERC20(_token);
        insurancePool = InsurancePool(_pool);

        _setChainlinkToken(_chainlinkToken);
        oracle = _oracle;
        jobId = stringToBytes32(_jobId);
        fee = 0.1 * 10**18;
    }

    struct Policy {
        address passenger;
        string flightNumber;
        uint256 departureTime;
        uint256 premiumAmount;
        uint256 payoutAmount;
        bool isTriggered;
        bool isSettled;
    }

    mapping(bytes32 => Policy) public policies;
    mapping(bytes32 => bytes32) public oracleRequests;

    event PolicyCreated(bytes32 indexed policyId, address indexed passenger, string flightNumber);
    event FlightStatusChecked(bytes32 indexed policyId, uint8 status);
    event PolicyPaidOut(bytes32 indexed policyId, uint256 amount);

    error InvalidTime();
    error InvalidAmount();
    error InvalidId();
    error FailedSistem();

    function stringToBytes32(string memory source) internal pure returns (bytes32 result) {
        bytes memory tempEmptyStringTest = bytes(source);
        if (tempEmptyStringTest.length == 0) {
            return 0x0;
        }
        assembly {
            result := mload(add(source, 32))
        }
    }

    function buyPolicy(string calldata flightNumber, uint256 departureTime, uint256 premiumAmount) external returns(bytes32 policyId) {
        if (departureTime <= block.timestamp + 1 days) revert InvalidTime();
        if (premiumAmount == 0) revert InvalidAmount();

        policyId = keccak256(abi.encodePacked(msg.sender, flightNumber, departureTime));
        if (policies[policyId].passenger != address(0)) revert InvalidId();

        uint256 payout = premiumAmount * 5;

        bool success = token.transferFrom(msg.sender, address(this), premiumAmount);
        if (!success) revert FailedSistem();

        policies[policyId] = Policy({
            passenger: msg.sender,
            flightNumber: flightNumber, 
            departureTime: departureTime,
            premiumAmount: premiumAmount,
            payoutAmount: payout,
            isTriggered: false,
            isSettled: false
        });

        emit PolicyCreated(policyId, msg.sender, flightNumber);

        return policyId;
    }

    function requestFlightStatus(bytes32 policyId) external {
        Policy storage policy = policies[policyId];
        if (block.timestamp <= policy.departureTime + 2 hours) revert InvalidTime();
        if (policy.isTriggered) revert FailedSistem();

        Chainlink.Request memory req = buildChainlinkRequest(jobId, address(this), this.fulfillRequest.selector);
        req.add("get", "ТУТ_БУДЕТ_URL_API");
        req.add("path", "status");

        bytes32 requestId = sendChainlinkRequest(req, fee);
        oracleRequests[requestId] = policyId;
        policies[policyId].isTriggered = true;
    }

    function fulfillRequest(bytes32 _requestId, bytes32 _status) internal override {
        bytes32 policyId = oracleRequests[_requestId];
        Policy storage policy = policies[policyId];
        if (!policy.isSettled) revert FailedSistem();

        uint256 flightStatus = uint256(_status);

        if (flightStatus == 0) {
            require(token.transfer(address(insurancePool), policy.premiumAmount), FailedSistem());
        }
        else {
            insurancePool.payOutInsurance(policy.passenger, policy.payoutAmount);
        }

        policy.isSettled = true;

        emit PolicyPaidOut(policyId, policy.payoutAmount);
    }
}
