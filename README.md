# Deployment Dependencies Permutations

This document outlines the various permutations of deployment dependencies in Solidity smart contracts, using examples from the Caller contract.

## P1: Direct Contract Type Use
State variable with Contract type + Constructor parameter of Contract type. This is the most straightforward dependency, where both the state variable and constructor parameter use the concrete contract type.

```solidity
Callee callee2; // State variable declared as contract type

constructor(..., Callee _callee2, ...) {
    callee2 = _callee2; // Direct assignment with no casting
}
```

## P2: Address Cast to Contract in Constructor
State variable with Contract type + Address parameter + Cast in constructor. This common pattern allows flexibility in passing addresses while maintaining type safety in the contract.

```solidity
Callee callee; // State variable declared as contract type

constructor(address _callee, ...) {
    callee = Callee(_callee); // Address cast to contract type in constructor
}
```

## P3: Runtime Casting of Address
State variable as address + Address parameter + Cast in runtime method. The address is stored as an address type but is cast to a contract type when needed at runtime.

```solidity
address callee3; // State variable declared as address type

constructor(..., address _callee3, ...) {
    callee3 = _callee3; // Stored as address
}

function setXFromAddress(uint256 _x) public {
    Callee(callee3).setX(_x); // Cast to contract type at runtime
}
```

## P4: Address Used in Low-Level Calls
State variable as address + Address parameter + Used in low-level calls. Even without explicit casting, this creates a deployment dependency.

```solidity
address callee4; // State variable declared as address type

constructor(..., address _callee4, ...) {
    callee4 = _callee4; // Stored as address
}

function lowLevelCall(uint256 _x) public {
    (bool success, ) = address(callee4).call(
        abi.encodeWithSelector(Callee.setX.selector, _x)
    ); // Low-level call to address
    require(success, "Low level call failed");
}
```

## P5: Interface Type with Address Cast
State variable with Interface type + Address parameter + Cast in constructor. This pattern provides flexibility while enforcing the interface contract.

```solidity
ICallee callee5; // State variable declared as interface type

constructor(..., address _callee5, ...) {
    callee5 = ICallee(_callee5); // Address cast to interface type in constructor
}
```

## P6: Direct Interface Type Use
State variable with Interface type + Constructor parameter of Interface type. Similar to P1 but using an interface instead of a concrete implementation.

```solidity
ICallee callee6; // State variable declared as interface type

constructor(..., ICallee _callee6, ...) {
    callee6 = _callee6; // Direct assignment with no casting
}
```

## P7: Address Cast to Interface at Runtime
State variable as address + Address parameter + Cast to interface at runtime. Similar to P3 but casting to an interface instead of a concrete contract.

```solidity
address callee7; // State variable declared as address type

constructor(..., address _callee7) {
    callee7 = _callee7; // Stored as address
}

function setXFromCallee(uint256 _x) public returns (uint256) {
    uint256 x = ICallee(callee7).setX(_x); // Cast to interface type at runtime
    return x;
}
```

## Additional Notable Case: Address in Modifiers
Address state variables used in modifiers also create dependencies, as the contract logic relies on them.

```solidity
address callee4; // State variable declared as address type

modifier onlyCallee4() {
    require(msg.sender == callee4, "Only callee4 can call this function");
    _;
}
```

In all these cases, the rule of thumb applies: "If a contract needs another contract's address to function, it creates a deployment dependency." This means the referenced contract must be deployed before the current contract for proper functionality.


    "this-contract" =  {
        "deployment_dependencies": [
            {
                state_var_name: "callee",
                state_UDC: "Callee",
                was_addr_to_udc: false
            },
            {
                ...
            }
        ]

    }


get all state variables in this-contract
    /// DIRECTS & CASTS IN INSTANSIATION - P1, P6, P2, P5 ///
    # - P1 & P6 will also catch the case when the state variable is type UDC but an address is passed in the constructor || initializer and casted
    # so P1 & P6 also implictly handle: P2, P5
    if variable type is user defined contract (UDC) & UDC defined in src/ or /contracts:
        # P1: Direct Contract Type Use
        if UDC != Interface (slither_fully_implmented??):
            if state variable -> left_side.expression (AssignmentOperation) in constructor || initializer:
                this-contract.deployment_dependencies.add({state-variable.name, UDC, false})
        
        # P6: Direct Interface Type Use
        if UDC == Interface (!slither_fully_implmented??):
            if state variable -> left_side.expression (AssignmentOperation) in constructor || initializer:
                this-contract.deployment_dependencies.add({state-variable.name, UDC, false})

    /// CASTS AT RUNTIME (POST DEPLOYMENT) - P3, P7  ///
    # Note: Since P4 & "Address in Modifiers" are never casted - we will need a one off solution for these, so lets hold off on these implementations for now

    if variable type is address:
        # Need the check below to ensure its a dependency
        if state variable -> left_side.expression (AssignmentOperation) in constructor || initializer:
            for f in contract.functions_declared:
                for n in f.nodes:
                    for ir in n.irs:
                        if isinstance(ir, TypeConversion) and isinstance(ir.type.type, Contract) and isinstance(ir.type, StateVariable) and ir.name == state_variable.name:
                            if ir.type.type defined in src/ or /contracts:
                                this-contract.deployment_dependencies.add({state-variable.name, ir.type.type.name, true})


    


    if variable type is address: 



    # notes
    - Parse our solodit repos for names of modifiers/addr to collect actors


    run time: 
    Missing Post-Constructor Assignment Detection
        Your pseudocode assumes that all state variables are assigned in the constructor or initializer. However, some contracts might set address dependencies in separate functions after deployment (although these would create runtime rather than deployment dependencies).