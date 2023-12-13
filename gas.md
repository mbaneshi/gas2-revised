Certainly, I'll provide your original text without any spell or typo corrections:

```markdown
# Manage and develop a strategy for gas
<details>
  <summary>

  ## Abstract
  </summary>

**Who is the target audience**:

Experienced FunC developers who already know FunC basics

**What is the purpose:**

The TON smart contract gas model is very unique and quite different from the EVM model. Contract developers must design a gas strategy. If they don't do this well, the contract can run out of TON balance for rent and be removed. Messages that the contract sends might not have enough gas and be rejected. So the purpose of this article is to help FunC developers manage gas and develop a strategy for gas usage to prevent pitfalls.

- I will write an article to help FunC developers write better smart contracts and avoid gas handling mistakes 
- I will discuss the general idea and motivation behind TON's gas model for smart contracts
- I will explain gas-related staff in detail and explain what's the logic behind them.
- I will list pitfalls where developers can make a gas mistake 
- I will deal with best practices (do's and don'ts) for handling gas in FunC.
- I will provide multiple hands-on code examples for clarification.
- I will provide some reference links for further reading 
</details>
<details>
  <summary>
  
## General Idea :
</summary>


 ## Introduction

Gas in TON is both an indicator of efficiency and a guideline and guard line of development processes. It may be times you think about a solution that may result in high gas consumption. This is a symptom of revising your solution and studying more to figure out how this novel system works as a whole.
You may conduct a solution that has a problem with the principle of who will be in charge of payment. If you plan to have a million users
but one smart contract is in charge of all payments, this may be a fault.
In TON, we must pay for both data and instruction; it is obvious
that putting less data and utilizing smooth simple instruction, in fact, is saving money. It is intentionally designed for light
overhead, resulting in speed and scalability.
So it is not a surprise if you hear smart contract contests awarded with low gas consumption.

To demonstrate why gas consumption in TON is an indicator of efficiency, we are going to inspect two versions of one famous
smart contract.
Version 5 of Toonkeeper wallet in contrast to version 4. This upgrade has 93% lower storage fees, just by utilizing offloading
the code into a shared library on Masterchain.

Gas price, rent price, and forwarding fees are necessary to protect the network against spam and reward validators for keeping consensus
running. This is of the utmost importance because TON is a public resource shared by everyone. It is our duty as developers to make
efficient use of this resource and produce the most optimal software possible.

Till now, in the TON document, there is some content addressing gas-related issues.

For example, [here](https://docs.ton.org/develop/smart-contracts/fees) gas was introduced, and transaction fees were discussed. By reading it, we
can figure out what elements of transaction fees are, what formula they utilize for calculation, and how storage fees will be calculated.

And [here](https://docs.ton.org/develop/howto/fees-low-level), provided a comprehensive overview of the low-level fees associated with transactions on the TON blockchain. The information covers transactions and phases, computation fees, gas costs, TVM instructions cost, FunC constructions gas fees, and fee calculation formulas for storage, forwarding, and actions.

This detailed documentation is valuable for developers and users who want to understand the inner workings of fee structures on the TON blockchain, especially when dealing with FunC code and low-level interactions.

In addition to two previous resources, there is a dedicated directory with some useful information [here](https://docs.ton.org/develop/smart-contracts/guidelines).
It is recommended you first read those resources and then come back here since this article writes upon those concepts.

### How is this text organized? :

How we can address one indicator of efficiency(gas) in such a complex system (TON), without knowing how this system works?
Well, I think it is impossible or at least incomplete.


  The blockchain industry has suffered from one significant problem for years known as Scalability. 
TON, Solve a significant problem, known as scalability.
There is a new and novel perspective on this solution that should be considered.  

We have some years of experience from the past, for example, we  have lessons from bleeding hard forks, so we and design systems around reconfigurability parameters.

TON is not just blockchain. It means TON is infrastructure, TON is the platform.
we have had some concepts in computer science for years, for example, actor model in functional programming like Erlang, we have stack machine concept in Forth, and sharding concept in database management as well.
We have async programming as well. The admiration comes from the fact that putting everything from the past together and developing paradigm-changing tools to solve these day's problems like ownership of data that must belong to individuals and not giant IT companies.

We consider gas in this text as a pivotal engineering concept, using it we explore parts of  the system.

</details>

<details>
  <summary>
    
  ## Origin of Gas :

</summary>

### Introduction

To develop our strategy for managing gas in the smart contract, it is vital to figure out how the TON blockchain works
and especially how gas plays its role in the system as a whole.

 ## General Idea

We break down our problem into some parts:

- How we can categorize gas consumption
- Where does environmental gas take place and what factors involved
- What are techniques or tricks for gas-saving improvement
- Which factors are involved in gas consumption
- Who is responsible for payment in any case
- When an error arises what happens and how to prevent them
- General idea about gas optimization mainly related to the architecture aspect

All gas-related staff belong to the TON Virtual Machine (TVM).TVM is where the smart contract gas fees took place.

For ease of handling the subject, we divided gas-related staff into three portions.

- gas storage fee
- gas computation fee
- gas communication fee.
  ![gas](assets/gas-partition-diagram.png)

For This, we discuss:

- we need to bring some technical details to the table to digest what is going on.

Let's start with storage:

- All data and instructions are packed as Cells. Cells have a hash as a representation. The cell can reference to Cells previously made.

- Both data and instruction Can be calculated as a bit sequence.
- Formula for storage depends on time delta and how many bits are involved.

For Computation, we need to consider these facts:

- every operation in TVM has specific gas fee usage.
- Jumping is expensive
- dealing with heavy calculations not recommended
- TVM has assembly language with the capability for extensibility


For communication, we need to consider this fact:

- every smart contract is responsible for sending an outbound message fee.
- we have two separate worlds known as internal and external
- Every world about communication has its own rules
-

So we need to investigate in every part:

- how gas will be calculated?
- What is the process of accomplishment
Certainly! Here's your exact text with the spell or typo mistakes corrected:

### Communication

#### External Messages:

1. **Gas Limit for External Messages:**
   - Initially set to gas_credit (10k gas).
   - Developers use `accept_message` to set the gas limit. Failure to do so may result in discarding the message if gas_credit is reached.
   - After completion, full computation fees are deducted based on the new gas limit.

2. **Handling Errors in External Messages:**
   - If an error occurs after `accept_message`, the transaction is written to the blockchain, fees are deducted, but storage is not updated, and actions are not applied.

#### Internal Messages:

1. **Gas Limit for Internal Messages:**
   - Set to `message_balance/gas_price` by default, with the message paying for its processing.
   - Contract can use `accept_message/set_gas_limit` to change the gas limit during execution.

2. **Bounceable and Non-Bounceable Internal Messages:**
   - Bounceable messages are bounced back if the destination contract doesn't exist or encounters an unhandled exception.
   - Non-bounceable messages are necessary for certain actions like creating new accounts.

### Design Pattern - Who is in charge?

- Creation of external messages is paid from the contract's balance.
- Use `tvm.rawReserve` before creating external messages to account for potential costs.

### Conclusion - Optimizing Gas Usage in TON Smart Contracts

1. **Fine-Tune Gas Limits:**
   - Carefully analyze computational requirements and set appropriate gas limits.

2. **Strategic Use of `ACCEPT` Instruction:**
   - Utilize the `ACCEPT` instruction judiciously, especially for additional gas required in external message processing.

3. **Gas Credit Management:**
   - Efficiently manage gas credits, considering unused gas from previous computations.

4. **Exception Handling Strategies:**
   - Implement robust exception handling to address out-of-gas scenarios and ensure transaction integrity.

5. **External Message Optimization:**
   - Use `ACCEPT` instruction to dynamically allocate gas for processing external messages.

6. **Cryptographic Operations Efficiency:**
   - Optimize cryptographic operations to enhance overall contract efficiency.

7. **Security Measures with Cryptography:**
   - Strengthen smart contract security through secure key management, proper usage of digital signatures, and robust hash functions.
