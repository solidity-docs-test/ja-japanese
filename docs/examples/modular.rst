.. index:: contract;modular, modular contract

*****************
Modular Contracts
*****************

.. A modular approach to building your contracts helps you reduce the complexity
.. and improve the readability which will help to identify bugs and vulnerabilities
.. during development and code review.
.. If you specify and control the behaviour or each module in isolation, the
.. interactions you have to consider are only those between the module specifications
.. and not every other moving part of the contract.
.. In the example below, the contract uses the ``move`` method
.. of the ``Balances`` :ref:`library <libraries>` to check that balances sent between
.. addresses match what you expect. In this way, the ``Balances`` library
.. provides an isolated component that properly tracks balances of accounts.
.. It is easy to verify that the ``Balances`` library never produces negative balances or overflows
.. and the sum of all balances is an invariant across the lifetime of the contract.

モジュラーアプローチでコントラクトを構築すると、複雑さを軽減し、読みやすさを向上させることができ、開発やコードレビューの際にバグや脆弱性を特定するのに役立ちます。各モジュールの動作を個別に指定して制御する場合、考慮しなければならない相互作用はモジュールの仕様間のものだけで、コントラクトの他のすべての可動部分ではありません。以下の例では、コントラクトは ``Balances``   :ref:`library <libraries>` の ``move`` メソッドを使用して、アドレス間で送信された残高が期待したものと一致するかどうかをチェックしている。このように、 ``Balances`` ライブラリはアカウントの残高を適切に追跡する独立したコンポーネントを提供しています。 ``Balances`` ライブラリが負の残高やオーバーフローを決して生成せず、すべての残高の合計がコントラクトのライフタイムにわたって不変であることを簡単に確認できる。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    library Balances {
        function move(mapping(address => uint256) storage balances, address from, address to, uint amount) internal {
            require(balances[from] >= amount);
            require(balances[to] + amount >= balances[to]);
            balances[from] -= amount;
            balances[to] += amount;
        }
    }

    contract Token {
        mapping(address => uint256) balances;
        using Balances for *;
        mapping(address => mapping (address => uint256)) allowed;

        event Transfer(address from, address to, uint amount);
        event Approval(address owner, address spender, uint amount);

        function transfer(address to, uint amount) external returns (bool success) {
            balances.move(msg.sender, to, amount);
            emit Transfer(msg.sender, to, amount);
            return true;

        }

        function transferFrom(address from, address to, uint amount) external returns (bool success) {
            require(allowed[from][msg.sender] >= amount);
            allowed[from][msg.sender] -= amount;
            balances.move(from, to, amount);
            emit Transfer(from, to, amount);
            return true;
        }

        function approve(address spender, uint tokens) external returns (bool success) {
            require(allowed[msg.sender][spender] == 0, "");
            allowed[msg.sender][spender] = tokens;
            emit Approval(msg.sender, spender, tokens);
            return true;
        }

        function balanceOf(address tokenOwner) external view returns (uint balance) {
            return balances[tokenOwner];
        }
    }

