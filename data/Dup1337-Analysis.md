The overall system design is wisely constructed whereas the core functions are called at the end of long call traces.

While this long call trace is enough to complete the validations before updating any mapping or positions, this brings a tradeoff where the readability drops significantly, and further development might be tampered with due to the structure being over-nested.

We respected the TradFi approach of the protocol a lot, however, having a different design (like Diamond Proxy Pattern) while holding the same approach might bring more core value in terms of development and the code size that the protocol is actually trying to reduce, especially considering the dependent protocol (Pendle) uses the same pattern.

As we already submitted this one as a separate issue, we want to underline that third-party approvals should be inlined in the function calls since assuming the dependent protocols will remain solid might be an optimist approach.


We saw that the Gas Optimization doesn´t suffice considering the length of call traces. While this might be the intended design or business decision to shift the users to L2, having a large gas cost on L1 wouldnẗ magnet the users considering some functions result in 3M  gas usage.

We encourage having more clear and frequent NATSPEC comments for almost every function. By this, the protocol could gain more bug-spotting power. 

We encourage reconsidering double oracle setup. As described in one of our submissions, manipulation of UniswapV3 TWAT (Time Weighted Average Tick) can be used as DoS vector for the protocol, especially on L2s.

We encourage the Wiselending team to extend automated tests to every chain that the protocol will run on. There are multiple places, where the logic differs between chains, and chains themselves have different ways of working and levels of EVM equivalence. However, the automated tests presented were meant to be run on main net.

Code style, while promotes abstractions and is easy to read, is also hard to understand and reason about. The human brain can effectively only hold around 7 items in short memory and is unsuited for deep reasoning highly scattered knowledge. Multiple levels of inheritance, cross-referencing contracts, and using function and storage pointers make it extremely hard to understand the code on a broader level and take a lot of time. Additionally, it breaks all automated IDE tools for tracing code flow. And time-to-understand the codebase is crucial during time-bounded security review, as it may be a factor differentiating getting rekt or not. 


### Time spent:
120 hours