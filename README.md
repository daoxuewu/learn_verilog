# Verilog 學習紀錄

參考教材: 
* [TT小教室-從零開始輕鬆學會Verilog](https://youtube.com/playlist?list=PLuhWBQnV46Q-3oMz33PFSqjhJOvDDVUbJ&feature=shared)

## 簡介
:::spoiler 為什麼需要**Verilog**?
* 數位電路是一堆邏輯閘(logic gate)、存儲器(Memory)、暫存器(Flip-Flop)和連線(wire)、匯流排(bus)與輸入輸出(IO)組成。
* 再透過 Place  & Route 工具轉化成晶圓廠(例如:台積電)能製作的 laytout 檔(GDS)
(附帶一提，IC設計公司把GDS上傳給晶圓廠的動作稱為 Tapeout，有人翻譯為下線或是流片)
* 如果沒有一種程式語言可以描述這些邏輯閘或是 register 的關係與連線，只能靠圖形連接來實現，這樣稍微複雜一點的電路馬上就看暈頭了，而 verilog 就是可以描述這些關係與連線的程式語言，用軟體來描述硬體，英文叫 Hardware Description Language
* 另外還有一種語法叫 VHDL，但是在台灣和美國的業界還是比較習慣使用 Verilg，所以這邊就先學 Verilog 就夠用了
:::

:::spoiler 那 RTL 又是什麼?
* 人類的大腦不適合寫像邏輯閘(低階)那樣的 Verilog
* 比較適合寫抽象或高階一點的電路模型
* 之後再利用合成工具(例如:Synopsys 的 Design Compiler 或是 Cadence 的 Genus)將高階的電路模型轉換成低階的邏輯閘
* 我們稱這種高階的電路模型為 RTL(Register-Transfer Level)
* 例子:
    ```verilog
    // Gate level netlist (新手階段netlist只要會看就行)
    module Full_Adder (A, B, Cin, Sum, Cout);
        input A, B, Cin;
        output Sum, Cout;
        wire w1, w2, w3;

        xor xor1(w1, A, B);
        and and1(w2, w1, Cin);
        and and2(w3, A, B);
        xor xor2(Sum, w1, Cin);
        or or1(Cout, w2, w3);
    endmodule

    // 上面和下面的程式碼都是符合 Verilog 語法的歐!

    // RTL
    module Full_Adder (A, B, Cin, Sum, Cout);
        input A, B, Cin;
        output Sum, Cout;

        assign {Cout, Sum} = {1'b0, A} + B + Cin;
    endmodule
    ```
:::
    
## 基本架構
#### 首先要先來建立 Verilog code 的 big picture
* 寫 Verilog RTL 很像是我們小時候在玩的堆積木。把一些不同形狀大小和功能的積木推成自己想要的樣子，有的積木是自己設計的，有的積木是在網路上找到別人已經設計好的，也有一些公司專門靠賣這些形形色色的積木賺錢，而一個積木就是 verilog 中的一個模塊(module)，而積木和積木是靠 input 與 output 相連，然後相同大小形狀功能的積木可以一直被創造(術語叫instantiate，實例化)，積木也可以一層包一層，層層堆疊(稱為hierarchical design)，工程師都是從小電路開始設計起，慢慢把小電路組成一個大電路，越來越複雜也包含越來越多功能。
* RTL檔案的副檔名通常是.v (就是代表 verilog 的 v)，每個小電路都是由 Verilog 的保留字 "**module**" 開始，直到 "**endmodule**" 結束
    >[!Tip]
    所以打開一個.v檔時第一件事情就是找它的 module 和 endmodule 在哪裡，建議每一個.v檔中只包含一個小電路，雖然 Verilog 沒有規定一個.v檔裡只能有一個 module，但業界的習慣通常還是會拆開來寫在不同的.v檔裡面，這樣比較方便管理
* 例子: 在module 後面到左括號前的那些字，就是這個 module 的名稱，我們叫 module name，大家也習慣直接用 module name 當檔名 
    ```verilog
    module Full_Adder ();
    ...
    ...
    endmodule
    ```
    >[!Warning] 注意
    >==Verilog 語法是有區分大小寫的(case sensitive)==，大寫的 F 的 Full_Adder 和小寫的 f 的 full_adder 會被當成不同的 design 喔!
    養成一個良好的 coding style 是很重要的!尤其是對於這種分大小寫的語言!
    
* 讓我們從一個簡單的範例開始學習吧!
![image](https://hackmd.io/_uploads/rkPXpa7Ulx.png)

    可以看到上圖中左邊的小積木，是一個簡單的 module 叫 Full_Adder，它有3個 inputs A, B 和上一位的進位 C in，還有2個 outputs 分別是 Sum 與給下一位的進位 C out 這個積木的功能就是把 A 加上 B 和上一位的進位 C in 來產生這位的和 Sum 與下一位的進位 C out。
    
    我們有了這個 Full_Adder 的積木後就可以開始拼出更多位數的加法器，像右邊的大積木是一個8位數的加法器，instantiate 的語法是先寫 module name，例如 Full_Adder 然後再寫 instance name (像是 u_FA0 到 u_FA7)，且須注意==同一個 module 內的 instance name 是不可以重複的，會造成語法錯誤===，這段 code 就不能被 compile
    
* **input** 用來宣告電路的輸入訊號，**output** 用來宣告電路的輸出訊號
![image](https://hackmd.io/_uploads/r1CLGAmLle.png)
    :::warning
    在 Verilog 裡面分號是代表這行結束的意思，一定要記得加歐，不然也會造成編譯錯誤
    :::
    上圖中常見的兩種寫法裡，列表式稍微麻煩一點，同一個名字要寫兩次，寫越多次犯錯的機會就越高，右邊的直接宣告式寫法比較輕鬆一點只要寫一次，但須注意在最後一個宣告的時候不要加逗點。
    
    另外，一個電路的 output 是不可或缺的，一個電路如果沒有輸出等於是一個沒有用的電路，因為外面的人根本不知道裡面發生什麼事，在跑模擬的時候可能看不出來，但實際在跑合成時如果沒有輸出訊號的話整個電路可能都會被化簡掉。
    
    附帶一提，這些輸入輸出業界術語稱 Port 或是 Pin，如果還要細分 Port 和 Pin 的話，習慣上最 design 的輸入輸出會叫 Port，其他層的輸入輸出用 Pin，不過很多人都是混著講，沒那麼講究~
    
    除了 input 和 output 以外還有一種型態叫 in out，就是這個 Port 既可當輸入也可當輸出，但這種形式不常見，也很少見，這邊就先不探究。

* 除了 input 和 output 以外，Verilog 還有兩種資料型態 **wire** 和 **reg**，這是用來描述接線和暫存器用的，訊號需要接上組合邏輯(combinational logic)時，就把訊號宣告成 **wire**，搭配 **assign** 語法使用，成為 combinational logic 的訊號線，或者是 instance 之間的接線也是用 **wire** 形式，而 **reg** 是 register 的縮寫，通常都是用在 **always** 語法裡，但要記得宣告為 reg 不代表會合成出 Flip-Flop 喔 (這是新手最容易搞不清楚的地方，之後會詳記介紹 **assign** 和 **always** 的用法，到時就會知道 wire 和 reg 怎麼區別了)

    範例 : 
    ![image](https://hackmd.io/_uploads/H1dTOR7Leg.png)
    input 的宣告方式一定是 wire 的形式，所以像圖上這樣寫的預設就是 wire，不用特別去修改成 input wire A 之類的，但 output 可以是 wire 或是 reg 所以說如果你的 output 想要宣告成 reg 的話，就必須像上面那樣寫成 output reg Sum 這種樣式 (總之，不寫 reg 就是預設用 wire，要用 reg 的就要特別寫出來)
    sum_int 和 cout_int 是右圖電路裡的2條線，用來連結 combinatiuonal logic A+B+Cin 的結果，這個電路再把這兩條線連接到兩個 registers Sum 和 Cout，而這兩個 registers 的 output 也直接連到這個 module 的 output 上，就像右圖上畫的這樣。
    :::warning
    切記，因為 Verilog 是硬體描述語言，硬體就是一個有實體的東西，因此寫 code 時心中一定要有辦法對應到硬體的樣子，千萬不要用軟體的思維來寫 code，這樣做出來會四不像，有時甚至不能合成，有錯時也會很難 debug
    :::

:::spoiler Verilog 的所有保留字
宣告 module name input 和 output 還有 wire 和 reg 等時都要避免使用這些字喔，雖然看起來一大堆但真正常用的沒多少個，常用的已經有 Highlight 起來了，等後面學會這些字的用法，大概也就清楚整個 Verilog 的語法了!
![image](https://hackmd.io/_uploads/rJz430QLeg.png)
:::

## array & assign
