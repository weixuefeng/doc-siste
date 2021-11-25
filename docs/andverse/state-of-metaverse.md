[!TOC]


# decentraland 状态

## 1. ownership layer, 所有权层

### Land 智能合约

Land eth 地址:`0xF87E31492Faf9A91B02Ee0dEAAd50d51d56D5d4d`, 遵循 ERC721 标准，每个 parcel 包括:
```
owner: 对该 Land 有绝对控制权
approval: 授权一个地址权限，可以转让和修改数据
updateOperator: 可以修改NFT数据 
assetData: 与地块有关的额外信息
```

还有:
```
operator:address允许转移任何代币
ping:address与此合约最后一次交互的 UNIX 时间戳（以秒为单位）
```

### Estate 合约，地产合约
地产合约，可组合的不可替代合约。将许多 LAND 放到一起，目的：用户在转让，出售，管理土地的时候可以节省 GAS。

### 去中心化交易市场
市场合约：`0x8e5660b4ab70168b5a6feea0e0315cb49c8cd539`
出价合约: `0xe479dfd9664c693b2e2992300930b00bfde08233`
设计目的: 
1. 使用 MANA 安全的，无需信任的交易地块
2. 交易可能在不依赖矿工以外的第三方的情况下发生。
3. 合约不收取任何费用，如果有收取费用，那么就需要销毁 MANA
4. 作为以太坊公民，合约应该准备好租用存储空间，而不是浪费存储空间


### 其他合约
1. 燃烧合约：销毁接受到的 ERC20 Token
2. 拍卖合约：用户 LAND 的第二次拍卖
3. Passthough: 可以将某些函数加入黑名单，安全防护
4. Vesting：根据时间释放 token合约

## 2. 分发和验证内容
1. 每个 NFT 需要有 `URL` 和 `CID`,根据这些从 IPFS 下载内容以便渲染到场景中；
2. 用户代理(浏览器)连接到内容服务，下载所需要的内容副本

分发逻辑:
1. 判断地块有没有 `CID` 信息
2. ? 判断该地块有没有其他签名信息

验证逻辑:
- 包裹信息中包括清单文件、签名信息(spec256k1)

1. 地址是否是地块的 owner
2. 该地址是否是地块的 operator
3. 地块是否是地产的一部分
4. 地址是否是地产的 owner
5. 地址是否是地产的 operator

### [Decentraland's IPFS Content Server HTTP API](https://github.com/decentraland/content-service)

## 3. 加载周边包裹算法:
全局状态包括:
- position: 玩家位置
- LoadedScenes: 已经加载的场景集合 或者 loading
- WorldMapper: 将 LAND 坐标映射到收到的最新且有效的场景清单的服务。它还避免了重叠/错误部署的场景。

算法分三步:
1. 拉取并调节场景信息，然后加载
    - 令 `Coordinates` 为距位置可见半径内所有地块的坐标集
    - 令 `ScenesToLoad` 成为一组空的场景
    - 对于坐标中的每个 `C` 
        - 设场景清单 `S` 为 `WorldMapper.scene(C)` 
        - 将 `S` 添加到 `ScenesToLoad`

2. Select which scenes should be loaded:
```
For every S in ScenesToLoad
    If S ∉ LoadedScenes:
    Load scene S
    Add S to LoadedScenes
```

3.  Load and unload scenes
```
For every S in LoadedScenes
    If S ∉ ScenesToLoad
    Unload scene S
    Remove S from LoadedScenes
```

## 4. 包裹清单
每个场景都需要许多配置参数和输入信息，以便用户代理可以正确显示场景并连接到其他客户端。这些字段包括其他所需材料（模型、脚本、纹理、声音……）的 IPFS CID 以及客户端应该如何渲染和行为的策略（是否允许飞行，客户端可能出现的安全传送区域是什么，使用哪个通信服务器与他人通信......）

eg:
```json
{
  "assets": {
    "scene.js": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b",
    "material_wood": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b",
    "material_iron": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b",
    "door_model": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b",
    "favicon_asset": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b",
    "background_music": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b",
    "snapshot": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b",
    "lod50": "/ipfs/QmZ1d5v83XSM7xbKDVAdFgS1PZmCzxuF5UtYrjMtFt6d6b"
  },

  "contact": {
    "name": "John Doe",
    "email": "john@doe.com"
  },

  "main": "scene.js",

  "scene": {
    "base": "2,2",
    "parcels": [
      "1,2", "1,3", "1,4",
      "2,2", "2,3", "2,4",
      "3,2", "3,3", "3,4"
    ]
  },

  "communications": {
    "type": "commsv2",
    "server": "https://comms.decentraland.org"
  },

  "spawnPoints": [
    {
      "position": {
        "x": 1.3, "y": 1.4, "z": 18.34
      },
      "rotation": {
        "x": 0, "y": 180, "z": 0
      },
      "default": true
    },
    {
      "area": {
        "x": [1.3, 2.5],
        "y": [0, 0],
        "z": [20, 20]
      }
    }
  ],

  "policy": {
    "contentRating": "E",
    "fly": "yes",
    "voiceEnabled": "yes",
    "blacklist": ["0,0", "1,0", "1,1", "1,1"],
  },

  "tags": ["land", "decentraland", "parcel"],

  "display": {
    "title": "My Land",
    "favicon": "favicon_asset",
    "preview": {
      "snapshot": "snapshot",
      "loadingColor": "#15ffae",
      "lod50": "lowres"
    }
  }
}

```

## 5. Scripting Scenes, 脚本场景
包裹清单文件包括一个名为 `main`的字段，它应该指向一个由每个用户代理同时执行的 javascript 文件。

```javascript
// Create a group to track all entities with a Transform component
const myGroup = engine.getComponentGroup(Transform)

// Define a System
export class RotatorSystem implements ISystem {
  // The update function runs on every frame of the game loop
  update() {
    // The function iterates over all the entities in myGroup
    for (let entity of myGroup.entities) {
      const transform = entity.get(Transform)
      transform.rotate(Vector3.Left(), 0.1)
    }
  }
}

// Add the system to the engine
engine.addSystem(new RotatorSystem())

// Create an entity
const cube = new Entity()

// Give the entity a transform component
cube.add(new Transform({
    position: new Vector3(5, 1, 5)
  }))

// Give the entity a box shape
cube.add(new BoxShape())

// Add the entity to the engine
engine.addEntity(cube)

```

## 6. Supporting dApps & Tooling
- [Marketplace](https://marketplace.decentraland.org)
[github](https://github.com/decentraland/marketplace)

- [Agora](https://agora.decentraland.org) 链下投票
Agora is an off-chain voting tool that can be used with any ERC-20 token that has been used to poll the community on different high-impact decisions for the protocol.

- [Builder](https://builder.decentraland.org)
builder 是一个易于使用的点击式装饰工具，它允许任何人快速构建场景、创建稍后可以用代码修改的布局，以及通过与他人共享项目进行协作。

- [命令行界面](https://github.com/decentraland/cli)
该cli用于Decentraland允许开发人员创建的场景，查询blockchain对不同土地的当前状态，并部署到内容服务器。

- [decentraland-ui](https://ui.decentraland.org) 和 [decentraland-dapps](https://github.com/decentraland/decentraland-dapps)