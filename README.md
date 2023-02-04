namesc  = "RANDOM TEN v2"
user    = "Alejandro"
bot     = "wdb" --wdb for web dice bot
py_mode = true
--if py_mode true, format > ║-=¦ Bet: [ BB: x ══ Chance: x ══ HI : x > x ] ❎  (Alejandro)

resetstats() resetseed()
if (string.lower(bot) == "wdb") then resetall() else resetchart() end

maxbal  = balance
minbet  = 1e-8
basebet = maxbal/1e6
if (basebet < minbet) then
    basebet = minbet
end
nextbet = basebet

bypass    = false
auto      = true
res_on_mp = true

res_ifprofit = 3e-8

kits = {{
    name = "Delicious",
    ch   = {1,3,5,9},--4
    dec  = {{10,15},{15,30}},
    inc  = {{15,20},{15,25}},
    res  = {
           on_win = {
                divmaxbal = 100, 
                decrease  = 0, 
                auto      = "off"
           },
           on_lose = {
                ls        = 10,
                divmaxbal = 500,
                decrease  = 0,
                auto      = "on"
           }
         }
    },{
    name = "Long and Medium Risk",
    ch   = {0.5,5,10,15},--4
    dec  = {{20,20},{10,30}},
    inc  = {{8,10},{11,17}},
    res  = {
           on_win = {
                divmaxbal = 100,
                decrease  = 0,
                auto      = "off"
           },
           on_lose = {
                ls        = 10,
                divmaxbal = 500,
                decrease  = 0,
                auto      = "on"
           }
         }
    },{
    name = "Risk",
    ch   = {10,20,40,60},--4
    dec  = {{30,30},{10,20}},
    inc  = {{40,40},{90,100}},
    res  = {
           on_win = {
                divmaxbal = 100,
                decrease  = 0,
                auto      = "on"
           },
           on_lose = {
                ls        = 30,
                divmaxbal = 500,
                decrease  = 0,
                auto      = "off"
           }
         }
    }
}

mp,mpp,count_ls,cprofit = 0,0,0,0

x      = 3
stx    = x
chance = kits[x].ch[4]

maxdrop  = 0
maxdropp = 0

function fix_num(num)
    n = tostring(num)
    if not string.match(n,"%.") then
        n = n.."."
    end
    if (num < 10) then
        n = "0"..n
    end
    if (string.len(n) == 4) then
        n = n.."0"
    elseif (string.len(n) == 3) then
        n = n.."00" 
    end
    return n
end

function substract_amount(cc,dec,bet)
    bet_dec  = {}
    win_dec  = {80,70,60}
    lose_dec = {70,50,30}
    if (bet) then
        bet_dec = win_dec
    else
        bet_dec = lose_dec
    end
    if (cc < 5) then
        return bet_dec[1]
    elseif (cc < 10) then
        return bet_dec[2]
    elseif (cc < 20) then
        return bet_dec[3]
    else
        return dec
    end
end

function set_bb_cc(cc,p_amount,kit,xx)
    bb = 0
    if (cc == kit[xx].ch[4]) then
        cc = kit[xx].ch[math.random(1,2)]
        bb = p_amount - (p_amount * (math.random(kit[xx].dec[1][1],kit[xx].dec[1][2])/100))
    elseif (cc == kit[xx].ch[3]) then
        cc = kit[xx].ch[2]
        bb = p_amount - (p_amount * (math.random(kit[xx].dec[2][1],kit[xx].dec[2][2])/100))
    elseif (cc == kit[xx].ch[2]) then
        cc = kit[xx].ch[1]
        if (math.random(100)%3 == 0) then cc = kit[xx].ch[4] end 
        bb = p_amount + (p_amount * (math.random(kit[xx].inc[1][1],kit[xx].inc[1][2])/100)) 
    else
        cc = kit[xx].ch[math.random(2,3)]
        bb = p_amount + (p_amount * (math.random(kit[xx].inc[2][1],kit[xx].inc[2][2])/100))
    end
    return {cc,bb}
end

function print_consolse(l)
    for i=1,#l do if (string.lower(bot) == "wdb") then log(l[i]) else print(l[i]) end end
end

function print_maxbet(msg)
    len_msg = string.len(msg)-11
    line    = "═"
    for l   = 1,len_msg do
        line = line.."═"
    end
    print(line.."\n"..msg.."\n"..line)
end

function dobet()
    success = "❎"
    hilo,hl,cc = "LO","<",lastBet.chance
    if (bethigh) then hilo,hl,cc = "HI ",">",(99.99 - lastBet.chance) end
    bethigh = math.random(50) > 10
    cprofit = cprofit + currentprofit
    if (win) then
        success = "✅"
        if (profit >= mp and res_on_mp or cprofit >= res_ifprofit and res_ifprofit > 0) then
            bypass   = false
            maxbal   = balance  
            mp       = profit
            mpp      = mp/(balance-mp)*100
            basebet  = balance/1e6
            nextbet  = basebet
            cprofit  = 0
            count_ls = 0
            x        = stx
            if (auto) then x = math.random(1,#kits) end
            chance = kits[x].ch[4]
        else
            on_off = string.lower(kits[x].res.on_win.auto)
            limit  = maxbal/kits[x].res.on_win.divmaxbal
            if (on_off == "on") then auto_res = math.random(50) > 25 end
            if (chance < kits[x].ch[4] and previousbet >= limit and limit > 0) then
                if (on_off == "off" or on_off == "on" and auto_res) then
                    bypass = false
                    if (kits[x].res.on_win.decrease > 0) then
                        substract = substract_amount(chance,kits[x].res.on_win.decrease,true)
                        nextbet   = previousbet - (previousbet * (substract/100))
                    else
                        nextbet = basebet
                    end
                    if (nextbet < minbet) then nextbet = minbet end
                    print_maxbet("║ Update Nextbet On Win: "..string.format("%.8f",nextbet).." / Auto: "..on_off.." ║")
                    sleep(15)
                end
            end
            count_ls = 0
        end
    else  
        count_ls = count_ls + 1
        on_off   = string.lower(kits[x].res.on_lose.auto)
        limit    = maxbal/kits[x].res.on_lose.divmaxbal
        tlose    = kits[x].res.on_lose.ls
        if (on_off == "on") then auto_res = math.random(50) > 25 end
        if (count_ls >= tlose and previousbet >= limit and limit > 0) then
            if (on_off == "off" or on_off == "on" and auto_res) then
                bypass = false
                if (kits[x].res.on_lose.decrease > 0) then
                    substract = substract_amount(chance,kits[x].res.on_lose.decrease,false)
                    nextbet = previousbet - (previousbet * (substract/100))
                else
                    nextbet = basebet
                end
                x = 3
                if (math.random(100)%5 == 0) then
                    x = 1
                end
                print("X: "..x.."")
                chance = kits[x].ch[math.random(2,3)]
                if (nextbet < minbet) then nextbet = minbet end
                print_maxbet("║ Update Nextbet On Lose: "..string.format("%.8f",nextbet).." / Auto: "..on_off.." / Ls: "..count_ls.." ║")
                count_ls = 0
                sleep(10)
            end
        end
    end
    if (bypass) then
        ch_nb   = set_bb_cc(chance,previousbet,kits,x)
        chance  = ch_nb[1]
        nextbet = ch_nb[2]
    else
        bypass = true
    end
    prof          = profit
    profp         = profit/(balance-profit)*100
    current_drop  = maxbal - balance
    current_dropp = current_drop/maxbal*100
    if (current_drop >= maxdrop) then
        maxdrop  = current_drop
        maxdropp = maxdrop/maxbal*100
    end
    if (nextbet < minbet) then nextbet = minbet end
    res    = fix_num(lastBet.roll)
    ccx    = fix_num(cc)
    ccy    = fix_num(lastBet.chance)
    s_user = ""
    if (py_mode) then s_user = "   ("..user..")" end
    l1     = "\n\n\n\n\t        ══════════════════════╗"
    l2     = " >_ SCRIPT     [   ¤  "..namesc.."  ¤   ]  >>  Kit: [ "..kits[x].name.." ]\n╔═════════════════════════════════╝"
    l3     = "║-=¦ Running by: ( "..user.." )\n║"
    l4     = "║-=¦ Bet: [ BB: "..string.format("%.8f",previousbet).."   ══   Chance: "..ccy.."   ══   "
    ..hilo..": "..res.."  "..hl.."  "..ccx.." ]  "..success..""..s_user..""
    l5 = "╚═════════════════════════════════════════════════════════════╝"
    l6 = "           { Current Profit }\t              {MaxProfit}"
    l7 = "  ╚═══════════════════╝      ╚═══════════════════╝"
    l8 = "       "..string.format("%.8f",prof).." / "..string.format("%.2f",profp).."%\t        "
    ..string.format("%.8f",mp).." / "..string.format("%.2f",mpp).."%"
    l9  = "\n           { Current Drop }\t              {MaxDrop}"
    l10 = "  ╚═══════════════════╝      ╚═══════════════════╝"
    l11 = "       "..string.format("%.8f",current_drop).." / "..string.format("%.2f",current_dropp).."%\t       "
    ..string.format("%.8f",maxdrop).." / "..string.format("%.2f",maxdropp).."%"
    l = {l1,l2,l3,l4,l5,l6,l7,l8,l9,l10,l11}
    if (py_mode) then l = {l4} end
    print_consolse(l)
end
