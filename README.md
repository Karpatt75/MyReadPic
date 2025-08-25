Тестоый мод, созданный ИИ

Код файла MyReadPic\config.cpp:
class CfgPatches 

    {  
	    class MyReadPic { 
		  units[] = {"MyPicturePaper"}; 
		  weapons[] = {}; 
		  requiredVersion = 0.1; 
		  requiredAddons[] = 
		    {
			  "DZ_Data",
			  "DZ_Scripts",
			  "DZ_Gear_Consumables"
		    }; 
	}; 
};

         class CfgMods 
		 { 
		 class MyReadPic 
		 { 
		 dir = "MyReadPic"; 
		 picture = ""; 
		 action = ""; 
		 hideName = 1; 
		 hidePicture = 1; 
		 name = "MyReadPic"; 
		 credits = ""; 
		 author = "you"; 
		 version = "1.0"; 
		 type = "mod"; 
		 dependencies[
		 ] 
		 =  {
			 "World",
			 "Mission"
			}; 
		 class defs 
		 { 
		 class gameScriptModule
        {
            value = "";
            files[] = {"$CurrentDir:MyReadPic/scripts/3_Game"};
        };
        class worldScriptModule
        {
            value = "";
            files[] = {"$CurrentDir:MyReadPic/scripts/4_World"};
        };
        class missionScriptModule
        {
            value = "";
            files[] = {"$CurrentDir:MyReadPic/scripts/5_Mission"};
        }; 
	}; 
};

         class CfgVehicles 
		 { class Paper; 
		 class MyPicturePaper: Paper 
		 { 
		 scope = 2; 
		 displayName = "Картинка на бумаге"; 
		 descriptionShort = "Прочтите, чтобы увидеть изображение."; 
		 model = "\dz\gear\consumables\t_note.p3d"; // стандартная модель Paper // при желании можно задать другие параметры, коллизии и т.п. 
		 }; 
};

class CfgScriptedMenus
{
    PictureViewerMenu = "PictureViewerMenu";
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

файл MyReadPic\textures\MyReadPic\paper_image.edds
