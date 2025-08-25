Тестоый мод, созданный ИИ

Код файла MyReadPic\config.cpp:
class CfgPatches
{
    class MyReadPic
    {
        units[] = {};
        weapons[] = {};
        requiredVersion = 0.1;
        requiredAddons[] = {"DZ_Data", "DZ_Gear_Consumables"};
        
        scripts[] = {
            "MyReadPic/scripts/3_Game/Defines", // ← ДОБАВЛЕНО
            "MyReadPic/scripts/4_World/myreadpic/items/MyPicturePaper",
            "MyReadPic/scripts/4_World/myreadpic/actions/ActionReadPicture", 
            "MyReadPic/scripts/5_Mission/myreadpic/gui/PictureViewerMenu"
        };
    };
};

class CfgMods
{
    class MyReadPic
    {
        type = "mod";
        
        class defs
        {
            class gameScriptModule
            {
                value = "";
                files[] = {"MyReadPic/scripts/3_Game"}; // ← ДОБАВЛЕНО
            };
            
            class worldScriptModule
            {
                value = "";
                files[] = {"MyReadPic/scripts/4_World"};
            };
            
            class missionScriptModule 
            {
                value = "";
                files[] = {"MyReadPic/scripts/5_Mission"};
            };
        };
    };
};

class CfgScriptedMenus
{
    PictureViewerMenu = "PictureViewerMenu";
};

class CfgVehicles 
{
    class Paper;
    class MyPicturePaper: Paper 
    { 
        scope = 2; 
        displayName = "Картинка на бумаге"; 
        descriptionShort = "Прочтите, чтобы увидеть изображение."; 
        model = "\dz\gear\consumables\t_note.p3d";
        
        // Основные параметры
        itemSize[] = {1, 1};
        weight = 10;
        varQuantityInit = 1;
        varQuantityMin = 0;
        varQuantityMax = 1;
        quantityBar = 0;
        canBeSplit = 0;
        stackedUnit = "";
        destroyOnEmpty = 0;
        
        // Визуальные настройки
        hiddenSelections[] = {"zbytek"};
        hiddenSelectionsTextures[] = {"MyReadPic\\textures\\MyReadPic\\paper_icon.edds"};
        
        // Звуки
        soundImpactType = "paper";
		
		inventorySlot = "Paper";
        itemBehaviour = 0;
        openable = 0;
        varWetMax = 0.3;
        
        // Действия
        class DamageSystem
        {
            class GlobalHealth
            {
                class Health
                {
                    hitpoints = 10;
                    healthLevels[] = {
                        {1.0,{"dz\gear\consumables\data\t_note.rvmat"}},
                        {0.7,{"dz\gear\consumables\data\t_note.rvmat"}},
                        {0.5,{"dz\gear\consumables\data\t_note_damage.rvmat"}},
                        {0.3,{"dz\gear\consumables\data\t_note_damage.rvmat"}},
                        {0.0,{"dz\gear\consumables\data\t_note_destruct.rvmat"}}
                    };
                };
            };
        };
	};
};

class CfgNonAIVehicles
{
    class ProxyAttachment;
    class ProxyPaper: ProxyAttachment
    {
        scope = 2;
        inventorySlot = "Paper";
        model = "\dz\gear\consumables\t_note.p3d";
    };
};


Код файла MyReadPic\scripts\3_Game\Defines.c :
static const int MYREADPIC_MENU_ID = 95001;

Код файла MyReadPic\scripts\4_World\MyReadPic\actions\ActionReadPicture.c :
class ActionReadPicture: ActionSingleUseBase
{
    void ActionReadPicture()
    {
        m_CommandUID = DayZPlayerConstants.CMD_ACTIONMOD_ITEM_ON;
        m_StanceMask = DayZPlayerConstants.STANCEMASK_ERECT | DayZPlayerConstants.STANCEMASK_CROUCH;
    }

    override void CreateConditionComponents()
    {
        m_ConditionItem = new CCINonRuined;
        m_ConditionTarget = new CCTNone;
    }

    override string GetText()
    {
        return "Прочитать";
    }

    override bool HasTarget()
    {
        return false;
    }

    override bool ActionCondition(PlayerBase player, ActionTarget target, ItemBase item)
    {
        if (!player || !item)
            return false;

        return (item == player.GetItemInHands() && MyPicturePaper.Cast(item) != null);
    }

    override void OnExecuteClient(ActionData action_data)
{
    if (!action_data || !action_data.m_Player)
        return;

    PlayerBase player = action_data.m_Player;
    ItemBase item = action_data.m_MainItem;

    MyPicturePaper paper = MyPicturePaper.Cast(item);
    if (!paper)
        return;

    string img = paper.GetPicturePath();

    UIManager uim = GetGame().GetUIManager();
    uim.EnterScriptedMenu(MYREADPIC_MENU_ID, null);

    UIScriptedMenu menu = uim.GetMenu();
    
    // Самый надежный способ DayZ - прямой вызов через рефлексию
    if (menu)
    {
        // Вызываем метод без проверки - если метод есть, он выполнится
        GetGame().GameScript.CallFunction(menu, "SetImage", new Param1<string>(img), null);
    }
}

    override void OnExecuteServer(ActionData action_data)
    {
        // Пустая реализация для сервера
    }
}

Код файла MyReadPic\scripts\4_World\MyReadPic\entities\MyPicturePaper.c :
class MyPicturePaper extends Paper 
{ // Путь к картинке, упакованной в мод 
string GetPicturePath() { return "MyReadPic\\textures\\MyReadPic\\paper_image.edds"; }

override void SetActions() 
{ super.SetActions(); AddAction(ActionReadPicture); } 
}

Код файла MyReadPic\scripts\5_Mission\MyReadPic\gui\PictureViewerMenu.c :
class PictureViewerMenu : UIScriptedMenu
{
    protected ImageWidget m_PictureImage;
    protected ButtonWidget m_CloseButton;
    protected string m_ImagePath;
    
    void PictureViewerMenu()
    {
        m_ImagePath = "";
    }
    
    override Widget Init()
    {
        layoutRoot = GetGame().GetWorkspace().CreateWidgets("MyReadPic/GUI/layouts/PictureViewer.layout");
        
        m_PictureImage = ImageWidget.Cast(layoutRoot.FindAnyWidget("PictureImage"));
        m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("CloseButton"));
        
        if (m_ImagePath != "" && m_ImagePath != "MyReadPic\\textures\\MyReadPic\\paper_image.edds")
        {
            SetImage(m_ImagePath);
        }
        
        return layoutRoot;
    }
    
    void SetImage(string imagePath)
    {
        m_ImagePath = imagePath;
        if (m_PictureImage && imagePath != "")
        {
            m_PictureImage.LoadImageFile(0, imagePath);
            m_PictureImage.Show(true);
        }
    }
    
    override bool OnClick(Widget w, int x, int y, int button)
    {
        if (w == m_CloseButton)
        {
            Close();
            return true;
        }
        return false;
    }
    
    override bool OnKeyPress(Widget w, int x, int y, int key)
    {
        if (key == KeyCode.KC_ESCAPE)
        {
            Close();
            return true;
        }
        return false;
    }
}

Код файла MyReadPic\scripts\5_Mission\MyReadPic\mission\MissionGameplay.c :
modded class MissionGameplay 
{ 
    override UIScriptedMenu CreateScriptedMenu(int id) 
    { 
        UIScriptedMenu menu = super.CreateScriptedMenu(id); 
        if (menu) return menu;

        switch (id)
        {
            case MYREADPIC_MENU_ID:
                return new PictureViewerMenu();
        }

        return null;
    } 
}

Код файла MyReadPic\GUI\layouts\PictureViewer.layout :
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<layout>
    <name>PictureViewerMenu</name>
    <menu_type>PictureViewerMenu</menu_type>
    <options>Scripted</options>
    
    <root>
        <widget>
            <name>Background</name>
            <type>ImageWidget</type>
            <pos>0 0</pos>
            <size>1 1</size>
            <alpha>0.5</alpha>
            <image>GUI/textures/black.edds</image>
            <flags>Exclusive|Background</flags>
        </widget>
        
        <widget>
            <name>MainPanel</name>
            <type>PanelWidget</type>
            <pos>0.2 0.1</pos>
            <size>0.6 0.8</size>
            <alpha>0.9</alpha>
            <image>GUI/textures/black.edds</image>
            <flags>Visible|Exclusive</flags>
            
            <widget>
                <name>TitleText</name>
                <type>TextWidget</type>
                <pos>0.05 0.02</pos>
                <size>0.9 0.08</size>
                <text>Просмотр изображения</text>
                <font>Bold</font>
                <textsize>24</textsize>
                <textcolor>FFFFFFFF</textcolor>
                <align>center</align>
                <flags>Visible</flags>
            </widget>
            
            <widget>
                <name>ImageContainer</name>
                <type>PanelWidget</type>
                <pos>0.05 0.12</pos>
                <size>0.9 0.75</size>
                <alpha>1.0</alpha>
                <image>GUI/textures/black.edds</image>
                <flags>Visible</flags>
                
                <widget>
                    <name>PictureImage</name>
                    <type>ImageWidget</type>
                    <pos>0.05 0.05</pos>
                    <size>0.9 0.9</size>
                    <alpha>1.0</alpha>
                    <flags>Visible</flags>
                    <image>MyReadPic\\textures\\MyReadPic\\paper_image.edds</image> <!-- Заглушка -->
                </widget>
            </widget>
            
            <widget>
                <name>CloseButton</name>
                <type>ButtonWidget</type>
                <pos>0.35 0.9</pos>
                <size>0.3 0.08</size>
                <text>Закрыть</text>
                <textsize>20</textsize>
                <textcolor>FFFFFFFF</textcolor>
                <flags>Visible</flags>
            </widget>
        </widget>
    </root>
</layout>

файл MyReadPic\textures\MyReadPic\paper_image.edds
