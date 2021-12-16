<!--
 * @Author: pony@diynova.com
 * @Date: 2021-12-15 11:21:23
 * @LastEditors: pony@diynova.com
 * @LastEditTime: 2021-12-15 13:47:30
 * @FilePath: /notes/docs/andverse/avatar.md
 * @Description: avatar 
-->
# Andverse 登录流程

## 1. 登录流程

1. 链接钱包，进行签名，**定义签名信息**，服务端根据签名信息获取 **资源** 信息，**资源**信息需要定义，第一次来进入 Default 设置，选择 **avatar**，完成之后进入 Play Scene。

## 2. avatar 定义：
1. 人物形象拆分: 躯干，眼睛，手臂，发行
```json
[
  {
    "timestamp": 1639548627660,
    "avatars": [
      {
        "userId": "0xf56339a5fc257244b35cecb54cba2546e39cba7d",
        "email": "",
        "name": "Nate#ba7d",
        "hasClaimedName": false,
        "description": "",
        "ethAddress": "0xf56339a5fc257244b35cecb54cba2546e39cba7d",
        "version": 2,
        "avatar": {
          "bodyShape": "urn:decentraland:off-chain:base-avatars:BaseMale",
          "snapshots": {
            "face": "https://peer.decentral.games/content/contents/QmZGnM3shNMcMj1wE5C341cZTFdRdquMJN6GvkkuSkycEE",
            "face128": "https://peer.decentral.games/content/contents/QmZEBAhPaoF7xQnR7pQFFoUoZd1t6BtFETQxvMmdLmrPgt",
            "face256": "https://peer.decentral.games/content/contents/QmVFktfdmdFipKqXm4WRBJdVsUWHu2HtEw6ySinWM88RvV",
            "body": "https://peer.decentral.games/content/contents/QmbaGp7WuSKiigjgUn5Gu7okvjKnBDzcoXmRx3u2G2CesY"
          },
          "eyes": {
            "color": {
              "r": 0.125,
              "g": 0.703125,
              "b": 0.96484375
            }
          },
          "hair": {
            "color": {
              "r": 0.98046875,
              "g": 0.82421875,
              "b": 0.5078125
            }
          },
          "skin": {
            "color": {
              "r": 0.94921875,
              "g": 0.76171875,
              "b": 0.6484375
            }
          },
          "wearables": [
            "urn:decentraland:off-chain:base-avatars:hair_coolshortstyle",
            "urn:decentraland:off-chain:base-avatars:eyebrows_00",
            "urn:decentraland:off-chain:base-avatars:eyes_11",
            "urn:decentraland:off-chain:base-avatars:mouth_06",
            "urn:decentraland:off-chain:base-avatars:hip_hop_joggers",
            "urn:decentraland:off-chain:base-avatars:turtle_neck_sweater",
            "urn:decentraland:off-chain:base-avatars:sport_black_shoes",
            "urn:decentraland:off-chain:base-avatars:black_sun_glasses"
          ]
        },
        "tutorialStep": 256,
        "interests": [],
        "unclaimedName": "Nate"
      }
    ]
  }
]
```



- wear
```json
{
  "wearables": [
    {
      "id": "urn:decentraland:matic:collections-v2:0x71abedc4e0f8c2f8739036d6d3f830f651e0d97e:0",
      "name": "MFT Style Classics",
      "description": "Classic Eyewear",
      "collectionAddress": "0x71abedc4e0f8c2f8739036d6d3f830f651e0d97e",
      "rarity": "epic",
      "i18n": [
        {
          "code": "en",
          "text": "MFT Style Classics"
        }
      ],
      "data": {
        "replaces": [
          "mask",
          "helmet"
        ],
        "hides": [],
        "tags": [],
        "representations": [
          {
            "bodyShapes": [
              "urn:decentraland:off-chain:base-avatars:BaseMale"
            ],
            "mainFile": "MFTGlasses.gltf",
            "contents": [
              {
                "key": "MFTGlasses.gltf",
                "url": "https://peer.decentral.games/content/contents/QmR28y5zAA7AW1XWUT1dAym36x1iyZWfGi2DyVZBfbAyg3"
              },
              {
                "key": "thumbnail.png",
                "url": "https://peer.decentral.games/content/contents/QmfBLf3JzDHbqJtpz3b9Pto2Z6rsq9gK4h7z55XMGbMaRg"
              }
            ],
            "overrideHides": [],
            "overrideReplaces": [
              "mask",
              "helmet"
            ]
          },
          {
            "bodyShapes": [
              "urn:decentraland:off-chain:base-avatars:BaseFemale"
            ],
            "mainFile": "MFTGlasses.gltf",
            "contents": [
              {
                "key": "MFTGlasses.gltf",
                "url": "https://peer.decentral.games/content/contents/QmR28y5zAA7AW1XWUT1dAym36x1iyZWfGi2DyVZBfbAyg3"
              },
              {
                "key": "thumbnail.png",
                "url": "https://peer.decentral.games/content/contents/QmfBLf3JzDHbqJtpz3b9Pto2Z6rsq9gK4h7z55XMGbMaRg"
              }
            ],
            "overrideHides": [],
            "overrideReplaces": [
              "mask",
              "helmet"
            ]
          }
        ],
        "category": "eyewear"
      },
      "image": "https://peer.decentral.games/content/contents/QmcJ3ibFqxhEpRBPDLzY7tXTdguPMADYDGrfuF4A64nPSB",
      "thumbnail": "https://peer.decentral.games/content/contents/QmfBLf3JzDHbqJtpz3b9Pto2Z6rsq9gK4h7z55XMGbMaRg",
      "contents": {
        "MFTGlasses.gltf": "QmR28y5zAA7AW1XWUT1dAym36x1iyZWfGi2DyVZBfbAyg3",
        "thumbnail.png": "QmfBLf3JzDHbqJtpz3b9Pto2Z6rsq9gK4h7z55XMGbMaRg"
      },
      "metrics": {
        "triangles": 470,
        "materials": 2,
        "textures": 2,
        "meshes": 2,
        "bodies": 2,
        "entities": 1
      },
      "createdAt": 1625771038316,
      "updatedAt": 1625771038316
    },
    {
      "id": "urn:decentraland:matic:collections-v2:0x954663cc1427c9ae6b63795bf78a888c06677608:0",
      "name": "Diamond Hands Hoodie",
      "description": "Join our HODL family > discord.gg/hodlhq",
      "collectionAddress": "0x954663cc1427c9ae6b63795bf78a888c06677608",
      "rarity": "epic",
      "i18n": [
        {
          "code": "en",
          "text": "Diamond Hands Hoodie"
        }
      ],
      "data": {
        "replaces": [],
        "hides": [],
        "tags": [
          "HODL",
          "DiamondHands",
          "HODLHQ",
          "IHWT",
          "inHODLweTrust"
        ],
        "category": "upper_body",
        "representations": [
          {
            "bodyShapes": [
              "urn:decentraland:off-chain:base-avatars:BaseMale"
            ],
            "mainFile": "male/HODL_HQ_Hoodie_-_Male (1).glb",
            "contents": [
              {
                "key": "male/HODL_HQ_Hoodie_-_Male (1).glb",
                "url": "https://peer.decentral.games/content/contents/QmVKBJDf3HMvMJnC41JsribQrCbEJrRkJNpLrhUSRGjGHR"
              }
            ],
            "overrideHides": [],
            "overrideReplaces": []
          },
          {
            "bodyShapes": [
              "urn:decentraland:off-chain:base-avatars:BaseFemale"
            ],
            "mainFile": "female/HODL_HQ_Hoodie_-_female.glb",
            "contents": [
              {
                "key": "female/HODL_HQ_Hoodie_-_female.glb",
                "url": "https://peer.decentral.games/content/contents/QmSTqSaenR9HvMRubgnUDbRAwPPVa8RkLAmrWSGAjCi2wu"
              }
            ],
            "overrideHides": [],
            "overrideReplaces": []
          }
        ]
      },
      "image": "https://peer.decentral.games/content/contents/QmSWMHmY6tmKGoAbM39xf7eNnvBxRQWQcKMDDTY5A3r8aF",
      "thumbnail": "https://peer.decentral.games/content/contents/QmZvX1Px9BrNhmCQuXQWoEstUeBoZvDh1qGV1qLJ35LJsP",
      "metrics": {
        "triangles": 1322,
        "materials": 2,
        "textures": 3,
        "meshes": 2,
        "bodies": 2,
        "entities": 1
      }
    },
    {
      "id": "urn:decentraland:matic:collections-v2:0xaaee4e0ea3de22dfc960a7f9c8bbd22f7081c5fa:1",
      "name": "MetaZoo Laurel Wreath",
      "description": "Laurel Wreath by MetaZoo International",
      "collectionAddress": "0xaaee4e0ea3de22dfc960a7f9c8bbd22f7081c5fa",
      "rarity": "rare",
      "i18n": [
        {
          "code": "en",
          "text": "MetaZoo Laurel Wreath"
        }
      ],
      "data": {
        "replaces": [],
        "hides": [],
        "tags": [
          "metazoo",
          "mz",
          "wreath",
          "tiara",
          "wearable"
        ],
        "category": "tiara",
        "representations": [
          {
            "bodyShapes": [
              "urn:decentraland:off-chain:base-avatars:BaseMale"
            ],
            "mainFile": "male/head_wreath_green.glb",
            "contents": [
              {
                "key": "male/head_wreath_green.glb",
                "url": "https://peer.decentral.games/content/contents/QmQVAN8P2YE3c8a8vWnHRhEVq9ey6LX4o5p76Gnm2XxaYM"
              },
              {
                "key": "male/thumbnail.png",
                "url": "https://peer.decentral.games/content/contents/QmfJrdFpX5sxBvqciTFJqN6HxAarsQqH3UqgstWAZDwBiv"
              }
            ],
            "overrideHides": [],
            "overrideReplaces": []
          },
          {
            "bodyShapes": [
              "urn:decentraland:off-chain:base-avatars:BaseFemale"
            ],
            "mainFile": "female/head_wreath_green.glb",
            "contents": [
              {
                "key": "female/head_wreath_green.glb",
                "url": "https://peer.decentral.games/content/contents/QmQVAN8P2YE3c8a8vWnHRhEVq9ey6LX4o5p76Gnm2XxaYM"
              },
              {
                "key": "female/thumbnail.png",
                "url": "https://peer.decentral.games/content/contents/QmfJrdFpX5sxBvqciTFJqN6HxAarsQqH3UqgstWAZDwBiv"
              }
            ],
            "overrideHides": [],
            "overrideReplaces": []
          }
        ]
      },
      "image": "https://peer.decentral.games/content/contents/QmfFA9i6GKHhaeLAtMhAkSHQZk8KGJPGPzbtnChT1497K5",
      "thumbnail": "https://peer.decentral.games/content/contents/QmfJrdFpX5sxBvqciTFJqN6HxAarsQqH3UqgstWAZDwBiv",
      "metrics": {
        "triangles": 96,
        "materials": 1,
        "textures": 2,
        "meshes": 1,
        "bodies": 1,
        "entities": 1
      },
      "createdAt": 1631294808590,
      "updatedAt": 1631294808590
    }
  ],
  "filters": {
    "wearableIds": [
      "urn:decentraland:matic:collections-v2:0xaaee4e0ea3de22dfc960a7f9c8bbd22f7081c5fa:1",
      "urn:decentraland:matic:collections-v2:0x954663cc1427c9ae6b63795bf78a888c06677608:0",
      "urn:decentraland:matic:collections-v2:0x71abedc4e0f8c2f8739036d6d3f830f651e0d97e:0"
    ]
  },
  "pagination": {
    "limit": 500
  }
}
```