# Polkadot-Auto-Nomination-Management-Tool


## Objective

to provide an auto-nomination-management tool to maximize dot rewards with given nomination configuration
<br />
<br />
## Brief Overview on Polkadot Staking

- **Reference** : [https://wiki.polkadot.network/docs/en/learn-staking](https://wiki.polkadot.network/docs/en/learn-staking)

- **Validator Selection**
    - 50~100 validators for each epoch(several hours) are selected by the NPoS election mechanism
    - NPoS election mechanism : calculating the backing DOTs for each mapping of nominator and validator by the blockchain
        - constraint : maximizing validator entrance hurdle
            - maximizing the amount of the backed dot of the smallest validator
        - nominator cannot decide the weights of each nomination towards each validator

- **Staking Rewards Distribution**
    - distribution to each validator pool
        - same amount of DOTs for each elected validator
    - distribution to each nominator in each validator-pool
        - proportional to the backing DOTs to the validator

- **Rewards Mechanism**
    - highlight : more stake on validator → less reward rate for nominator
        - nominators should distribute the nomination to validators with below characteristics
            - very high probability of validator election
            - as low stake on the validator as possible
    - claiming rewards : rewards are kept available for 84 eras(84 days)

- **Nomination Management**
    - unbonding takes 28 days
    - immediate nomination change is available
        - but is repeated nomination change in short time allowed?

- **Accounts**
    - Controller can manage nomination! (does not need stash account)
<br />
<br />

## Reward-Optimization Tool

- **Objectives**
    - any nominator needs dynamic nomination strategy to maximize the staking reward rate
    - especially nominators with large DOT holding will need the nomination strategy
    - a reward-optimization tool : automatically adjust nomination to maximize the staking reward rate

- **Setup and Security Conditions**
    - control key : a local server should have a control key of the nominator
    - node api endpoint : there should be a node api endpoint to receive information and broadcast txs
    - firewall : every inbound except ssh can be blocked

- **Configurations**
    - `validator_whitelist`
        - definition : list of validators who can be nominated
    - `min_stake`, `max_stake`
        - definition : min/max staking amount for each validator
    - `min_frequency`
        - definition : minimum period of time to pause between nomination adjustment execution
    - `min_valpool_percentage`, `max_valpool_percentage`
        - definition : min/max validator-pool size in percentage of total global staking amount
        - characteristics
            - too low min : too high risk of the validator not elected
            - too high max : too inefficient staking reward rate
    - `improvement_delta_threshold`
        - definition : a threshold for expected staking reward rate improvement by nomination adjustment
        - characteristics
            - too low threshold : unnecessarily too frequent nomination adjustment
            - too high threshold : too lazy to stay long in inefficient nomination distribution
    - `commission_rate_ignore`
        - definition : whether to ignore commission rate of each validator or not

- **Methodology**
    - general idea
        - the program finds the nomination distribution with highest reward rate under given configurations
        - the program then automatically generates and broadcasts nomination adjustment transactions
    - algorithm
        - it is found by brute-force simulation
        - to be improved on next version
    - roadmap
        - we will find better algorithm to achieve better precision and faster calculation time

- **Consideration on Execution of Nomination Target Change**
    - problem : Polkadot only allows multiple nominations in one stash account by below restriction
        - the weights of each nomination toward each validator are decided by the blockchain
        - the nominator has no right on deciding the weights of each nomination
        - it will result in very inefficient reward rate
            - nominator should utilize very precise diversification strategy to acquire highest possible reward rate, especially for holders with large DOT quantity
    - solution : split into N stash accounts
        - nominate each splitted account to different validators
        - the nominator can diversify nomination to different validators with self-decided weights
        - the precision of diversification will be limited to 1/N %
            - if N=20, possible nomination size for each validator is 0%, 5%, 10%, ... (of total asset)
            - if N=100, possible nomination size for each validator is 0%, 1%, 2%, ... (of total asset)

- **Operation and Security**
    - Automatic vs Semi-automatic
        - Automatic
            - the program calculates the best diversification weights
            - the program then automatically creates and broadcasts nomination change transactions
        - Semi-automatic
            - the program calculates the best diversification weights
            - the program then alerts fund manager and print out necessary nomination change transactions to be signed and broadcasted
            - the fund manager signs to the transactions and broadcasts to the network
    - Security
        - Automatic
            - it needs controller keys of each stash account to operate
            - the controller key cannot create send transaction → limited risk of key stealing
        - Semi-automatic
            - the program does not need any key at all
