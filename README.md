# Silver Sixpence Merkle Tree Generator

A Merkle Tree is a privacy-preserving data structure that uses hash proofs to store and manage large datasets. 
It uses these hash functions to construct layers of nodes that build a tree-like structure with several layers of decreasing size, until only a single node remains. This node is called the Merkle Root.
There is no industry standard on how to generate Merkle Trees, so many different customized versions may exist. 
The *Silver Sixpence Merkle Tree Generator* uses its own approach, but can easily be modified on several parameters based on customer demand. 

Our standard methodology is as follows:
The Silver Sixpence Merkle Tree Generator generates the leaf nodes by concatenating a client ID, a random salt, an Audit ID and asset balances. It then calculates a SHA256 hash on the result:

Client_ID: Unique client identifier. Can be an ID number or hash of the username

Example: ```Client_ID = "287e29b8-de5b-4924-8906-b216f2d48cd6"```

Client_Balances = (Asset1=Balance | Asset2=Balance | Asset3=Balance ...)

Example: ```Client_Balances = "BTC=76.83|ETH=19.26|XRP=68.95|USDC=6.32"```

Salt is a random string of arbitrary length to ensure the uniqueness of all leaves.

Example: ```Salt = "2A496ECE"```

Audit_ID is a unique identifier for the audit that took place.

Example: ```Audit_ID = "PORNOV22"```

Account_Record = concat(Account_ID + Salt + Audit_ID + Client Balances)

Example: ```Account_Record = "287e29b8-de5b-4924-8906-b216f2d48cd62A496ECE PORNOV22BTC=76.83|ETH=19.26|XRP=68.95|USDC=6.32"```

Leaf = SHA256(Account_Record)


Not every element mentioned above is necessary for every audit. Either a Salt or Audit_ID value is needed so that static Account Records do not hash to the same value during different audits, but if both values are included then it will add redundancy.

The Merkle Tree is constructed by concatenating groups of these leaf nodes together and hashing the result. The size of the group is called the Merkle Width and is typically 2. The height of the Merkle Tree refers to the number of layers between the leaves and the Root, and is a function of the size of the dataset and the Tree Width.

In the diagram below, the width of the tree is two and the height is 3. The dataset consists of 4 accounts:


![Image1](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/1_4leafTree.png)



The Silver Sixpence Merkle Tree generator also has the capability to generate trees with width 4, which lowers the total height of the tree. This is, however a non-standard methodology and only done upon customer request. 

![Image2](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/10_width2wifth4comparison.png)

If the number of leaves or nodes on a particular height is not a multiple of the tree width (i.e. #leaves(mod(width)) != 0), then the last node is duplicated and move up by one level, so that there are always 2 children nodes to every parent node.

The tree below was built from a dataset with 9 leaf nodes, so the last node had to be duplicated and moved from layer 4 to layer 3. But since Layer 3 also has an uneven amount of nodes, the last one is duplicated again and moved to layer 2. Layer 2 then ends up with 3 nodes, so the last node is duplicated one again and moves to layer 1. It is then hashed with its neighbor to produce the parent node of layer 0 (which is the Merkle Root in this case).



![Image4](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/11_9leafsTree.png)


We do single SHA256 hashes all the child nodes to produce parents.

We will create a number of dummy accounts to be included in the leaf nodes. We typically generate at least 10% dummy accounts by spawning a random customer ID data with a 0-balance. This is done to add additional privacy to the data.

Consider a dataset of 40 accounts. 10% (four) dummy accounts are generated with a zero asset balance and inserted randomly between all the existing accounts. The leaf nodes are then generated by hashing the concatenated string of account data. In the image below, these leaf nodes form the bottom layer of the tree. 
Each node is then grouped with 3 neighboring nodes, concatenated and hashed to build the next layer. This step is repeated until only a single node remains, the Merkle Root.


![Image5](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/12_BigWidth2Tree.png)

A user could verify that his/her account data was included in the total dataset that was included in the tree, by independently reproducing the Root from its account data.


# User Verification

Any exchange client could verify that his/her account data was included in the total dataset that was included in the tree, by independently reproducing the Root from its account data.

Let’s say Amoné has an account at fictitious Exchange *SixpenceCoin*. She logs into her account and notice that an independent Proof-of-Reserves audit has recently been done, and that *SixpenceCoin* has enough assets to cover all of their liabilities. Amoné wants to know if her Bitcoin balance was indeed included when the total liabilities were calculated. 

On her profile page, she has a tab that includes data from the audit. Amoné’s balances at the time of the audit (called the snapshot) is displayed, along with her unique *customer ID*, an identifier unique to the particular audit, a random number called *Salt* and a *Merkle Root*. Amoné decides to independently verify the auditor’s results. 
She goes through the following steps:

**Step 1: Create the Merkle Leaf**

Amoné first needs to build her Merkle Leaf, which is a hash her *account ID*, *Salt*, *Audit ID* and *Balances*. Each exchange or auditor may have their own preferred structure, but *SixpenceCoin* uses the following format:

*Customer_ID*: A hash of the client’s username

*Salt*: Pseudorandom hexadecimal string, trimmed to 16 characters

*Audit_ID*: POR + Month of Audit + Year of Audit

*Client_Balances*: (Asset1=Balance | Asset2=Balance | Asset3=Balance ...)

Example:
```
Customer_ID = “287e29b8-de5b-4924-8906-b216f2d48cd6”
Audit_ID = "PORNOV22"
Salt = "2A496ECE"
Client_Balances = "BTC=6.83|ETH=1.26|XRP=88.95|USDC=2.32"
```


Amoné sees that the audit snapshot was taken at 2022/11/24 23:59 and verifies that her balances at that time was indeed as indicated on her profile page. She then concatenates all of this data together to form her Account Record:

Account_Record = concat(Account_ID + Salt + Audit_ID + Client Balances)

```
Account_Record = "287e29b8-de5b-4924-8906-b216f2d48cd62A496ECE PORNOV22BTC=76.83|ETH=19.26|XRP=68.95|USDC=6.32"
```

Amoné copies the Account_Record string and heads over to  [https://silversixpence.co.za/](https://silversixpence.co.za/) where she pastes the string into the form and verifies the hash that SixpenceCoin displays in her profile. 

She then directs to the Auditors’ website [https://www.mazars.co.za/] (https://www.mazars.co.za/) and navigates to the *SixpenceCoin* November 2022 audit page.

Here, she is presented with a  field from which she can search for her Merkle Leaf hash. The web application builds her unique tree path with all the hashes to the Root. Amoné is now ready to do a Merkle-Proof.

**Step 2: Build the first Branch**

*SixpenceCoin* has 20 million clients, so a Merkle Tree that hashes 2 nodes would be 25 levels high. 

Amoné already has one of the two leaves required to build the first branch – the one that she produced herself. She therefore needs hashes from three of her neighboring accounts. These hashes, together with their sequence, are provided by the auditor on their web-application. Amoné concatenates the four leaf nodes in their correct order and hashes the result to produce the first branch.


![Image7](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/13_Node21Node22Tree.png)


**Step 3: Repeat**

Amoné now needs to repeat the process of concatenating nodes to build the next branch. The auditor will provide the neighboring child node for every level, which Amoné needs to concatenate to produce the parent node. This step is repeated until only a single node remains, which is called the Merkle Root. If she successfully managed to reproduce the public Merkle Root, then she can be certain that her asset balances were included in the total liabilities that were observed by the auditor.

The complete tree structure will look like the image below. Note that the auditor will only reveal the minimum amount of information for Amoné to do independent verification, which is the nodes highlighted in green below. The rest of the nodes are kept private, as they are not needed for full verification.


![Image8](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/14_Width2MerkleProof.png)

The auditor website displays Amoné’s proof as a tower of vertical hashes. This is simply done to provide a better user experience. Since the auditor stores the client balances, they might display that to Amoné as well, if the exchange gave them the necessary permission.


![Image9](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/15_exchangeFrontend.png)







