## Abstract

这是一个EIP-721的扩展协议标准。它提出了(`initialState`, `finalState`, `trigger`)方法帮助链上状态管理。

## Motivation
在业务场景下执行nft函数执行需要在特定状态下才可以成功，状态异常会导致合约执行失败甚至产生业务bug。nft如果不包含状态描述的接口则需要则需要编写大量的If/else语句，导致代码膨胀逻辑混乱。

引入(`initialState`, `finalState`, `trigger`)，可以帮助开发人员更好的抽象业务逻辑。

## Specification

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY" and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

```solidity
/**
 * @dev the EIP-165 identifier for this interface is 0x7a0cdf92.
 */
interface IERCxxxx /* is IERC721 */ {
    /**
     * @dev Emitted when tokenId's current state changed.
     *
     * Note that 'form' and 'to' must be different.
     */
    event StateChanged(uint256 indexed tokenId, uint8 from, uint8 to);

    /**
     * @dev Returns this nft's initial state.
     *
     * Requirements:
     *
     * - `tokenId` must exist.
     */
    function initialState(uint256 tokenId) external view returns (uint8);
    
    /**
     * @dev Returns this nft's final state.
     *
     * Requirements:
     *
     * - `tokenId` must exist.
     */
    function finalState(uint256 tokenId) external view returns (uint8);

    /**
     * @dev The abstract interface that triggers the state change.
     *
     * Requirements:
     *
     * - `tokenId` must exist.
     */
    function trigger(uint256 tokenId) external;
}
```
The `initialState(uint256 tokenId)` function MAY be implemented as `pure` or `view`.

The `finalState(uint256 tokenId)` function MAY be implemented as `pure` or `view`.

The `trigger(uint256 tokenId)` function MAY be implemented as `public` or `external`.

The `StateChanged` event MUST be emitted when a nft's current state is changed.

## Test Cases
```solidity
#import "../IERCxxxx.sol";

contract WeaponsDemo is IERCxxxx {
    enum State {
        Low,
        Middle,
        High
    }

    mapping(uint256 => State) currentState;

    constructor(string memory name, string memory symbol)
        IERCxxxx(name, symbol)
    {}

    function mint(uint256 tokenId, address to) public {
        _mint(to, tokenId);
        currentState[tokenId] = State.Low;
    }

    function power(uint256 tokenId) public returns(uint256) {
      if (currentState[tokenId] == State.Low) {
        return 10;
      } else if (currentState[tokenId] == State.Middle) {
        return 100;
      } else if (currentState[tokenId] == State.High) {
        return 1000;
      } 
      return 0;
    }

    function initialState(uint256) external view returns (State) {
      return State.Low;
    }

    function finalState(uint256) external view returns (State) {
      return State.High;
    }

    function trigger(uint256 tokenId) external {
      require(currentState[tokenId] != finalState(tokenId));
      currentState[tokenId] += 1; 
    }
}
```