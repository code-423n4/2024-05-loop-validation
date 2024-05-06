# Centralization 


### [C-01] SetOwner should have a 2 step ownership

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L336-L340


### [C-02] Malicious owner can change AllowToken without any delay 

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L360-L366




# QA/Low

### [L-01] Writing totalSupply += _amount would cleaner and simpler to write than totalSupply = totalSupply + _amount

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L180



### [L-02] Claimed amount can be manipulated with a direct deposit

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L262