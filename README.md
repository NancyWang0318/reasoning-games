# reasoning-games
一個關於推理的遊戲
import tkinter as tk
from tkinter import messagebox

# 遊戲資料結構
class Case:
    def __init__(self, description, choices):
        self.description = description
        self.choices = choices  # [(選項文字, 下一步key)]

# 案件與分支設計（多案件串連）
game_cases = {
    # 案件一：富豪家中失竊案（主線初現、副線真兇明確）
    'start': Case(
        '你是一名新晉偵探，接到第一個案件：富豪家中失竊。你要先去哪裡調查？',
        [
            ('現場勘查', 'scene'),
            ('詢問管家', 'butler'),
            ('詢問女僕', 'maid'),
        ]
    ),
    'scene': Case(
        '你在現場發現一個奇怪的腳印和一枚鈕扣。你要怎麼做？',
        [
            ('採集證據', 'evidence'),
            ('繼續搜尋', 'search'),
            ('詢問女僕', 'maid'),
        ]
    ),
    'butler': Case(
        '管家神色緊張，聲稱案發時在廚房。你要怎麼做？',
        [
            ('相信他', 'trust_butler'),
            ('繼續追問', 'press_butler'),
            ('檢查廚房', 'kitchen'),
        ]
    ),
    'maid': Case(
        '女僕表示案發時在打掃二樓，並提到曾看到管家深夜外出。你要怎麼做？',
        [
            ('詢問管家', 'butler'),
            ('調查二樓', 'upstairs'),
            ('詢問其他傭人', 'other_staff'),
        ]
    ),
    'evidence': Case(
        '你發現腳印與管家鞋子吻合，鈕扣屬於女僕制服。你要怎麼做？',
        [
            ('質問管家', 'press_butler'),
            ('質問女僕', 'press_maid'),
            ('調查其他傭人', 'other_staff'),
        ]
    ),
    'search': Case(
        '你找到一張神秘紙條，上面寫著「午夜花園」。你要怎麼做？',
        [
            ('前往花園', 'garden'),
            ('詢問管家', 'butler'),
            ('詢問女僕', 'maid'),
        ]
    ),
    'trust_butler': Case(
        '你選擇相信管家，但案件陷入僵局。',
        [
            ('重新開始', 'start'),
            ('詢問女僕', 'maid'),
            ('調查現場', 'scene'),
        ]
    ),
    'press_butler': Case(
        '管家終於坦白，案發時其實在花園。他說：「我只是去澆花，沒做壞事！」',
        [
            ('前往花園', 'garden'),
            ('繼續追問', 'butler_confess'),
            ('詢問女僕', 'maid'),
        ]
    ),
    'kitchen': Case(
        '廚房一切正常，沒有異狀。',
        [
            ('詢問管家', 'butler'),
            ('詢問女僕', 'maid'),
            ('調查現場', 'scene'),
        ]
    ),
    'upstairs': Case(
        '二樓有女僕遺落的手帕，並發現窗戶有被打開的痕跡。',
        [
            ('質問女僕', 'press_maid'),
            ('調查窗戶', 'window'),
            ('詢問其他傭人', 'other_staff'),
        ]
    ),
    'other_staff': Case(
        '其他傭人表示案發時只看到女僕和管家在一樓和花園間來回。',
        [
            ('詢問管家', 'butler'),
            ('詢問女僕', 'maid'),
            ('調查現場', 'scene'),
        ]
    ),
    'press_maid': Case(
        '女僕神色慌張，最終承認偷竊。她說：「我只是想幫家裡還債，沒想到會被發現……」',
        [
            ('逮捕女僕', 'maid_confess'),
            ('繼續調查', 'scene'),
            ('詢問管家', 'butler'),
        ]
    ),
    'window': Case(
        '窗戶外有泥土痕跡，顯示有人從這裡進出。',
        [
            ('調查花園', 'garden'),
            ('詢問女僕', 'maid'),
            ('詢問管家', 'butler'),
        ]
    ),
    'garden': Case(
        '你在花園發現失竊物品，並找到女僕的指紋。',
        [
            ('質問女僕', 'press_maid'),
            ('調查現場', 'scene'),
            ('詢問管家', 'butler'),
        ]
    ),
    'butler_confess': Case(
        '管家堅稱自己只是澆花，並未參與偷竊。',
        [
            ('調查花園', 'garden'),
            ('詢問女僕', 'maid'),
            ('調查現場', 'scene'),
        ]
    ),
    'maid_confess': Case(
        '女僕被逮捕。她坦白：「我家裡欠債，實在走投無路才會偷東西。對不起……」\n你在審問過程中發現女僕曾與一名神秘男子聯絡，這成為主線新線索。',
        [
            ('進入第二案件', 'case2_intro'),
        ]
    ),

    # 案件二：畫廊名畫失蹤案（副線真兇明確，三選項）
    'case2_intro': Case(
        '第二案件：著名畫廊的鎮館之寶失蹤。現場有一張與上一案相似的紙條。你要怎麼做？',
        [
            ('調查畫廊監視器', 'case2_camera'),
            ('詢問畫廊經理', 'case2_manager'),
            ('詢問保全', 'case2_guard'),
        ]
    ),
    'case2_camera': Case(
        '監視器畫面被人刪除，但你發現一個可疑身影。你要怎麼做？',
        [
            ('追查身影', 'case2_shadow'),
            ('詢問經理', 'case2_manager'),
            ('詢問保全', 'case2_guard'),
        ]
    ),
    'case2_manager': Case(
        '經理神色慌張，聲稱案發時在倉庫。你要怎麼做？',
        [
            ('相信他', 'case2_trust_manager'),
            ('繼續追問', 'case2_press_manager'),
            ('調查倉庫', 'case2_storage'),
        ]
    ),
    'case2_guard': Case(
        '保全表示案發時巡邏，並看到經理與一陌生男子交談。你要怎麼做？',
        [
            ('詢問經理', 'case2_manager'),
            ('追查陌生男子', 'case2_shadow'),
            ('調查監視器', 'case2_camera'),
        ]
    ),
    'case2_shadow': Case(
        '你追查到一名與上一案女僕有聯繫的男子。',
        [
            ('繼續追查', 'case2_suspect'),
            ('詢問經理', 'case2_manager'),
            ('詢問保全', 'case2_guard'),
        ]
    ),
    'case2_trust_manager': Case(
        '你選擇相信經理，但案件陷入僵局。',
        [
            ('重新開始第二案', 'case2_intro'),
            ('詢問保全', 'case2_guard'),
            ('調查監視器', 'case2_camera'),
        ]
    ),
    'case2_press_manager': Case(
        '經理坦白曾與神秘男子接觸。',
        [
            ('追查神秘男子', 'case2_suspect'),
            ('詢問保全', 'case2_guard'),
            ('調查監視器', 'case2_camera'),
        ]
    ),
    'case2_storage': Case(
        '倉庫內發現一只手套，上面有經理的指紋。',
        [
            ('質問經理', 'case2_confront_manager'),
            ('詢問保全', 'case2_guard'),
            ('調查監視器', 'case2_camera'),
        ]
    ),
    'case2_suspect': Case(
        '你發現神秘男子與經理有金錢往來紀錄。',
        [
            ('質問經理', 'case2_confront_manager'),
            ('追查神秘男子', 'case2_suspect_confess'),
            ('詢問保全', 'case2_guard'),
        ]
    ),
    'case2_confront_manager': Case(
        '經理終於承認偷畫。他說：「我欠下巨債，只能鋌而走險。那神秘男子是中間人。」',
        [
            ('逮捕經理', 'case2_manager_confess'),
            ('追查神秘男子', 'case2_suspect_confess'),
            ('詢問保全', 'case2_guard'),
        ]
    ),
    'case2_suspect_confess': Case(
        '神秘男子被捕，供出主線黑市組織的線索。',
        [
            ('進入第三案件', 'case3_intro'),
            ('重新審問經理', 'case2_manager'),
            ('調查畫廊現場', 'case2_intro'),
        ]
    ),
    'case2_manager_confess': Case(
        '經理被逮捕。他坦白：「我只是想還債，沒想到會被發現……」\n你獲得主線黑市組織的關鍵線索。',
        [
            ('進入第三案件', 'case3_intro'),
        ]
    ),

    # 案件三：神秘失蹤人口案（副線真兇明確，三選項）
    'case3_intro': Case(
        '第三案件：一名失蹤人口與前兩案有關聯。你要怎麼調查？',
        [
            ('調查失蹤者家中', 'case3_home'),
            ('詢問鄰居', 'case3_neighbor'),
            ('調查失蹤者公司', 'case3_company'),
        ]
    ),
    'case3_home': Case(
        '你在家中發現一張黑市交易的收據和一封威脅信。',
        [
            ('追查黑市', 'case3_blackmarket'),
            ('詢問家屬', 'case3_family'),
            ('調查失蹤者公司', 'case3_company'),
        ]
    ),
    'case3_neighbor': Case(
        '鄰居表示最近常有陌生人出入，且失蹤者與公司主管關係緊張。',
        [
            ('調查公司主管', 'case3_boss'),
            ('詢問家屬', 'case3_family'),
            ('調查家中', 'case3_home'),
        ]
    ),
    'case3_company': Case(
        '公司主管神色閃爍，聲稱失蹤者最近行為怪異。',
        [
            ('質問主管', 'case3_boss'),
            ('調查失蹤者辦公桌', 'case3_desk'),
            ('詢問同事', 'case3_colleague'),
        ]
    ),
    'case3_blackmarket': Case(
        '你發現失蹤者曾與黑市組織聯絡。',
        [
            ('追查黑市組織', 'case4_intro'),
            ('詢問家屬', 'case3_family'),
            ('調查公司', 'case3_company'),
        ]
    ),
    'case3_family': Case(
        '家屬表示失蹤者最近常接到恐嚇電話。',
        [
            ('調查電話紀錄', 'case3_phone'),
            ('調查家中', 'case3_home'),
            ('詢問公司主管', 'case3_boss'),
        ]
    ),
    'case3_boss': Case(
        '主管最終承認綁架失蹤者，動機是失蹤者發現其貪污。',
        [
            ('逮捕主管', 'case3_boss_confess'),
            ('調查黑市', 'case3_blackmarket'),
            ('詢問家屬', 'case3_family'),
        ]
    ),
    'case3_desk': Case(
        '你在辦公桌發現一份揭發主管貪污的證據。',
        [
            ('質問主管', 'case3_boss'),
            ('調查黑市', 'case3_blackmarket'),
            ('詢問同事', 'case3_colleague'),
        ]
    ),
    'case3_colleague': Case(
        '同事表示失蹤者最近壓力很大，懷疑主管有問題。',
        [
            ('調查主管', 'case3_boss'),
            ('調查家中', 'case3_home'),
            ('調查黑市', 'case3_blackmarket'),
        ]
    ),
    'case3_phone': Case(
        '電話紀錄顯示多次來自黑市組織的號碼。',
        [
            ('追查黑市組織', 'case4_intro'),
            ('調查主管', 'case3_boss'),
            ('詢問家屬', 'case3_family'),
        ]
    ),
    'case3_boss_confess': Case(
        '主管被逮捕。他坦白：「我怕東窗事發，只好鋌而走險……」\n你獲得主線黑市組織的進一步線索。',
        [
            ('進入第四案件', 'case4_intro'),
        ]
    ),

    # 案件四：黑市交易案（副線真兇明確，三選項）
    'case4_intro': Case(
        '第四案件：你潛入黑市，發現前幾案的失竊物品都在這裡。你要怎麼行動？',
        [
            ('潛伏觀察', 'case4_observe'),
            ('直接報警', 'case4_police'),
            ('假扮買家', 'case4_disguise'),
        ]
    ),
    'case4_observe': Case(
        '你發現幕後黑手正是畫廊經理與富豪管家聯手。',
        [
            ('設法蒐證', 'case4_evidence'),
            ('通知警方', 'case4_police'),
            ('假扮買家', 'case4_disguise'),
        ]
    ),
    'case4_police': Case(
        '警方行動過於倉促，黑手逃脫。',
        [
            ('重新潛入黑市', 'case4_intro'),
            ('潛伏觀察', 'case4_observe'),
            ('假扮買家', 'case4_disguise'),
        ]
    ),
    'case4_disguise': Case(
        '你假扮買家接觸黑市頭目，發現其與前案神秘男子有聯繫。',
        [
            ('設法蒐證', 'case4_evidence'),
            ('通知警方', 'case4_police'),
            ('潛伏觀察', 'case4_observe'),
        ]
    ),
    'case4_evidence': Case(
        '你成功錄下黑市頭目與經理、管家的交易證據。',
        [
            ('質問黑市頭目', 'case4_boss_confront'),
            ('通知警方', 'case4_police'),
            ('假扮買家', 'case4_disguise'),
        ]
    ),
    'case4_boss_confront': Case(
        '黑市頭目終於承認罪行。他說：「我只是替真正的幕後黑手辦事，真正的主謀另有其人！」',
        [
            ('逮捕黑市頭目', 'case4_boss_confess'),
            ('追查幕後黑手', 'case5_intro'),
            ('通知警方', 'case4_police'),
        ]
    ),
    'case4_boss_confess': Case(
        '黑市頭目被逮捕。他坦白：「我只是收錢辦事，真正的主謀你們很快就會知道……」\n你獲得主線最終黑手的明確線索。',
        [
            ('進入第五案件', 'case5_intro'),
        ]
    ),

    # 案件五：真相大白（主線大真兇，三選項，多重結局）
    'case5_intro': Case(
        '第五案件：你掌握所有證據，準備揭露幕後黑手。你要怎麼行動？',
        [
            ('公開指控', 'case5_accuse'),
            ('設下陷阱', 'case5_trap'),
            ('秘密跟蹤', 'case5_follow'),
        ]
    ),
    'case5_accuse': Case(
        '你公開指控，黑手否認並試圖反咬你。',
        [
            ('堅持證據', 'case5_goodend'),
            ('選擇妥協', 'case5_badend'),
            ('設下陷阱', 'case5_trap'),
        ]
    ),
    'case5_trap': Case(
        '你設下陷阱，成功錄下黑手自白。',
        [
            ('交給警方', 'case5_goodend'),
            ('公開指控', 'case5_accuse'),
            ('秘密跟蹤', 'case5_follow'),
        ]
    ),
    'case5_follow': Case(
        '你秘密跟蹤黑手，發現其與前案所有真兇都有聯繫。',
        [
            ('設下陷阱', 'case5_trap'),
            ('公開指控', 'case5_accuse'),
            ('堅持證據', 'case5_goodend'),
        ]
    ),
    'case5_goodend': Case(
        '你成功將幕後黑手繩之以法，真兇身份明確——原來是富豪的親信律師張文傑，操控一切只為奪取遺產！\n他坦白：「我為富豪家族賣命多年，卻始終得不到應有的回報。這一切，只是為了奪回屬於我的東西……」\n你成為傳奇偵探，正義得以伸張！',
        [
            ('重新開始遊戲', 'start'),
        ]
    ),
    'case5_badend': Case(
        '你選擇妥協，黑手逍遙法外，正義未能伸張。\n主線真兇身份：富豪的親信律師張文傑，動機為奪取遺產與權力。',
        [
            ('重新開始遊戲', 'start'),
        ]
    ),
}

class DetectiveGame:
    def __init__(self, root):
        self.root = root
        self.root.title('推理冒險遊戲')
        self.text = tk.Label(root, text='', wraplength=400, justify='left', font=('微軟正黑體', 14))
        self.text.pack(pady=20)
        self.buttons = []
        self.current_case = 'start'
        self.show_case(self.current_case)

    def show_case(self, case_key):
        case = game_cases[case_key]
        self.text.config(text=case.description)
        for btn in self.buttons:
            btn.destroy()
        self.buttons = []
        for choice_text, next_key in case.choices:
            btn = tk.Button(self.root, text=choice_text, width=30, font=('微軟正黑體', 12),
                            command=lambda k=next_key: self.show_case(k))
            btn.pack(pady=5)
            self.buttons.append(btn)

if __name__ == '__main__':
    root = tk.Tk()
    root.geometry('500x400')
    app = DetectiveGame(root)
    root.mainloop()
