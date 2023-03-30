//LiteLoaderScript Dev Helper
/// <reference path="c:\Users\zjn18\Desktop\开发\LLBDS\Lib/dts/HelperLib-master/src/index.d.ts"/> 

ll.registerPlugin(
    /* name */ "DChatGPT",
    /* introduction */ "无需任何配置即可使用的ChatGPT",
    /* version */ [0,1,1],
    /* otherInformation */ {
        "作者": "HaiPaya"
    }
); 


var PlayerData = new KVDatabase(`.\\plugins\\DChatGPT\\PlayerData`)

function ui(pl) {
    let fm = mc.newSimpleForm()
    fm.setTitle("DChatGPT")
    if(PlayerData.get(pl.xuid)) {
        fm.addButton("关闭")
    }
    else {
        fm.addButton("开启")
    }
    pl.sendForm(fm, (pl, id) => {
        if(id == 0) {
            if(PlayerData.get(pl.xuid)) {
                PlayerData.set(pl.xuid, false)
            }
            else {
                PlayerData.set(pl.xuid, true)
            }
            ui(pl)
        }
    })
}

mc.regPlayerCmd("dchat", "DChatGPT", (pl) => {
    ui(pl)
})

mc.listen("onChat", (pl, msg) => {
    if(PlayerData.get(pl.xuid)) {
        network.httpGet(`https://chat.dzdgame.cn/sendmsg?id=${pl.xuid}&msg=${msg}`, (sta, res) => {
            if(sta == 200) {
                let dt = data.parseJson(res)
                if(dt.state != 200) {
                    pl.tell(`[DChatGPT] 后端错误!`)
                }
                else {
                    pl.tell(`[DChatGPT] 生成中...`)
                    let uuid = dt.uuid
                    let rs = setInterval(() => {
                        network.httpGet(`https://chat.dzdgame.cn/getmsg?uuid=${uuid}`, (sta, res) => {
                            if(sta == 200) {
                                dt = data.parseJson(res)
                                if(dt.state == 200) {
                                    pl.tell(`[DChatGPT] ${dt.msg}`)
                                    clearInterval(rs)
                                }
                            }
                            else {
                                pl.tell(`[DChatGPT] 无法连接至后端!`)
                                clearInterval(rs)
                            }
                        })
                    }, 1000 * 1)
                }
            }
            else {
                pl.tell(`[DChatGPT] 无法连接至后端!`)
            }
        })
    }
})
