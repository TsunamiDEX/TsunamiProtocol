# Tsunami Protocol - limit order book trading with sharable liquidity

<a href="https://discord.gg/bGF7eEAgdR"><img src="https://discord.com/assets/ff41b628a47ef3141164bfedb04fb220.png" width="250"></a>

[Official Discord Server](https://discord.gg/bGF7eEAgdR)

Project Name: Tsunami

Brief introduction: To achieve limit orderbooks on Chia Network and provide sharable fundamental liquidity.

## Project Overview

### Background
DeFi's total value locked has already reached 110 billion, while Ethereum occupies 78 billion. In a few years, DeFi has developed incredibly rapidly. During this process, primitive users have been suffering from Ethereum’s high gas fee, extremely low TPS and MEV problems. Until EIP1559 and some organizations like Flashbots appear, those problems are alleviated. Thanks to repeated attempts of early adopters, DeFi has taken shape.

Currently, DeFi is based on some smart contract platforms with Account Model such as Ethereum. Therefore, as it grows, performance bottlenecks inevitably emerge, resulting in a variety of Layer 2 solutions. Although Layer 2 solutions alleviate the performance and transaction friction problems of smart contract platforms, they break the composability in DeFi of Layer 1.

But why does no one pay attention to Bitcoin, the UTXO model that can almost expand capacity unlimitedly?

Bitcoin itself has limitations such as its Turing-incomplete scripting system and the low-efficiency of its POW consensus. Even though UTXO is powerful, with these limitations, it is difficult to implement the base function of DeFi.

Chia, however, makes all this possible. It makes Coin a first-class citizen and brings Chialisp, which is not as easy to understand as the account model of Ethereum, but this transformation makes UTXO-based DeFi a reality on Chia. As the superset of Lisp, Chialisp brings the advantage of Lisp into Chia’s Smart Coin. The coin set model and Chialisp complement each other.

### Vision for Tsunami Protocol
The foundation of Tsunami Protocol is based on Chialisp. Numerous computations are done off chain and verification are done on chain, which means Tsunami Protocol can almost expand capacity unlimitedly. Moreover, Smart Coin, based on Chialisp, can use formal verification to ensure security, which makes the contract more reliable and auditable.

Smart Coin can be considered as a coin contains a program, when you spend it, the program got executed. And Coins in other Chia DApps, also as first-class citizens, can be seamlessly compatible with Tsunami Protocol to achieve interoperability of fundamental liquidity.

From our perspective, Chia Coin Set Model, based on the design concept of UTXO, is a true electronic cash and programmable money. It is a "banknote" that exists independently in the virtual space. This inspired us to use first principle thinking to go back to the original starting point of "Exchange", to think about how an exchange should be conducted on DeFi, and to explore and experiment with shared fundamental liquidity on Chia Network.

## Project Detail

Tsunami Protocol is a trading protocol of decentralized orderbooks. Users can create orders, cancel orders, fill orders, and view order history in the frontend which they need to synchronize with Full Node. The whole transaction process is completely independent of any centralized node. Thanks to UTXO-based Coin Set Model and Chialisp, users can trade on Tsunami with near-centralized exchange speed with extremely low fees.

From a functional perspective, key implementation and standard definition of Tsunami Protocol are shown below.

![Figure 1](https://raw.githubusercontent.com/HashPocket/TsunamiProtocol/master/images/Figure_1.png)

![Figure 2](https://raw.githubusercontent.com/HashPocket/TsunamiProtocol/master/images/Figure_2.png)

1. Maker establishes Maker Puzzle and Taker Puzzle based on order information. Taker Puzzle contains order information, such as the address and order amount of Makers. Taker Puzzle Hash and related order information are included in the Maker Puzzle. Maker Coin (Its puzzle is called Maker Puzzle and amount is called Maker Amount) is generated when Maker uses the corresponding wallet (Standard Wallet or Colorful Wallet).
2. According to the public Tsunami Gateway Puzzle, Gateway Coin is generated temporarily. Order params are passed to gateway when you make a solution contains order data to spend the gateway coin.
3. The transaction is conducted through SpendBundle which is consisted of Gateway Coin Spend and Maker Coin Spend.
4. Broker monitors puzzles and coins that they are interested in on chain, gathers order information from the Solution of Tsunami Gateway Coin and update order status from the Solution of Maker Coin.
5. Broker maintains orderbooks, creates new orders, updates order status and cancels expired orders
6. Taker gets order information from the orderbooks.
7. Taker builds an ephemeral Taker Coin according to order information and spends Maker Coin and Taker Coin simultaneously.
8. Taker can consume it again, if it is still left in Maker Coin, which can recreate itself.

## Basic Order Structure

In Tsunami Protocol, each order is a unique puzzle with `genesis_maker_coin_id` as its order ID. The specific structure and meanings are shown below.

| Field Name                                                    | Comment                                                                                  |
| ----                                                          | ----                                                                                     |
| latest_maker_coin_id                                          | latest_maker_coin_id                                                                     |
| status                                                        | 0 initialization 1 Confirm on chain 3 Cancel 4 Partial filled orders 5 All filled orders |
| fee                                                           | service fee                                                                              |
| expired_in                                                    | expire time, timestamp of Unix, seconds                                                  |
| coin_amount                                                   | the rest amount of maker coin in current order                                           |
| taker_puzzle_has                                              |                                                                                          |
| taker_amount                                                  |                                                                                          |
| maker_amount                                                  |                                                                                          |
| maker_cc_id                                                   | If it is colorful coin，it is colorful id. If it is standard coin, it is "chia".         |
| taker_cc_id                                                   | same as maker_cc_id                                                                      |
| receiver_taker_coin_address                                   | receive the address of taker coin                                                        |
| receiver_maker_coin_address                                   | receive the address of maker coin                                                        |
| pubkey                                                        | authentication                                                                           |
| confirmed_at_index                                            |                                                                                          |
| created_at_time                                               |                                                                                          |
| genesis_maker_coin_id                                         | the parent coin name of first Maker Coin                                                 |

## Orderbook
The orderbook is a list of the order structure above. So far, orderbooks are established by Broker. In the future, we will index orderbooks and provide global orderbook apis for querying orders.

The orderbook is generated from parsing on-chain transactions. Let’s suppose that we start from a certain block, keep tracking the latest block, use its transactions_filter to check whether it contains Tsunami_GATEWAY_PUZZLE (there will be further introduction about this contract below) or Maker Puzzles and get all related coins. If a new coin with Tsunami_GATEWAY_PUZZLE shows up, which means there is a new order, the order status will be updated, and the new order will be saved in the local SQLite database.

### Create Orders
We will open-source Tsunami_GATEWAY_PUZZLE which can be called by any user. Actually, the order data is stored in the Solution. Considered of the extremely low transaction fees, Solution may contain a simple verification or a low commission to ensure the economical security, preventing from malicious attacks.

### Match Orders
Users can create their Taker Puzzle which contains order information, such as Maker’s address and amount, after querying the Solution of Maker Coin and Gateway Coin through Broker. Once the Spend Bundle, which is composed of Gateway Coin Spend and Maker Coin Spend, is broadcast to the network, and gets included in a block, the transaction is completed.

![Figure 3](https://raw.githubusercontent.com/HashPocket/TsunamiProtocol/master/images/Figure_3.png)

To fill the order, firstly, users must create Taker Coin according to the order information. And then spend both Maker Coin and Taker Coin to ensure the integrity of the transaction, mainly to ensure that the information of Taker Coin (transfer address, transaction price and amount) is correct. The address is embedded in the PUZZLE, and the transaction price and amount are submitted in Solution. In order to ensure proper execution of TAKER COIN, all these information is collected in the Message of CREATE_PUZZLE_ANNOUNCEMENT and use ASSERT_PUZZLE_ANNOUNCEMENT in Maker Coin.

In the aspect of against front-running, in a centralized exchange, once a user submits a market order, it will be filled. Therefore, front-running is a common concern in centralized exchange. But in Tsunami, the market price comes from a specific order that the driver module has already matched, and maker coin has been spent. Even if someone attempts to front run the order, the interests of the user will not be harmed, because the user can see the pre-execution result of his order before submitting it, and naturally will not submit it if it is unfavorable to him. Therefore, there is no guarantee that the front-runner will be able to "profit from the front-run", so the rational service provider will not risk it.

The Broker’s orderbook serves as a centralized service that provides users with extremely fast service, but without compromising the interests of Tsunami users. This is only possible due to Chia’s UTXO-like Coin Set Model and is not possible on chains that use global state machines such as Ethereum.

### Cancel Orders by Users

Users call the `cancel_by_me` function in maker coin which will transform the amount of maker coin to the `receiver_maker_coin_address`.

### Fill Orders

When users are placing orders, they do not know whether they play the role as a Maker, or the order is filled automatically. (Afterall, order information has been changing.) We will implement order-routing function in Tsunami SDK. If all orders are filled, it will execute taker module. If orders are partially filled, the rest will execute recreate_self function, and a new puzzle will be there. (A new order will appear.)

When users initiate a market order, firstly, they need to choose one order or multiple combined orders which meet the conditions in the orderbooks and merge all the transactions in one SpendBundle. Users sign transactions and wait until the order is filled. If there is not such an order, a limit order is generated and waits for other users to fill.

![Figure 4](https://raw.githubusercontent.com/HashPocket/TsunamiProtocol/master/images/Figure_4.png)

### Order History
Users can get their full transaction history by running the open-source Tsunami Client. In fact, Tsunami Client just help users filter the pubkey of orders. When the pub key of orders matches the users, then that order belongs to that user.
Also, by partnering with an SPV wallet like Nucle.io, users will be able to see their historical orders placed through the Tsunami Protocol directly within their usual wallet.

## Future Plans
### Price Oracle
With a certain amount of trading volume on Tsunami, time-weighted price oracle can be carried out with the help of Singleton pattern.

### Margin trade
Build a lending pool that Tsunami Protocol will automatically help users borrow/repay when they choose to trade with leverage. The user is trading on margin, and if a large price deviation occurs, it triggers a forced liquidation of the position.

### Derivative trade
Users can tokenize a stable coin like xUSD, carbon credits or $APPL on Tsunami Protocol. And they can synthesize and trade any asset on Tsunami Protocol. Also, users can mint NFTs to explore the metaverse on Chia Network.

### Research on Layer 2
* Focus on Infinite Scaling of Performance

In the future, after Tsunami reaches a certain trading volume and trading frequency, we can establish a Layer 2 network to aggregate several transactions and then send them together to the chia network. At the same time, orders of Maker can be shared among layer 2 nodes, to achieve the goal of sharing liquidity. Other projects can easily access Tsunami Protocol through the Driver module provided by us, and orders can also be routed to Tsunami Protocol for processing through the Tsunami Gateway.

## License
Apache Licence 2.0

## Additional Information
* What work has been done so far?

We have completed the MVP (Minimum Viable Product) and are building the frontend page and auditing the economical security.

* Are there any other teams doing the same kind of projects?

Not yet. If so, we are willing to communicate with them and collaborate to improve the underlying liquidity design for Chia Network.

* How to cooperate with Chia ecosystem?

We will try to simplify and abstract the puzzle design and make Tsunami Protocol a more fundamental component. And we will try to communicate more with other DeFi project designers to make the design more adaptive.

