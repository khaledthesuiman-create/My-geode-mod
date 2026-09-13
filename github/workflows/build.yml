#include <Geode/Geode.hpp>
#include <Geode/modify/MenuLayer.hpp>

using namespace geode::prelude;

// A global manager to handle our custom paginated toolbar
class ToolloaderManager {
public:
    static std::vector<CCMenuItemSpriteExtra*> registeredButtons;
    static int currentPage;
    static const int BUTTONS_PER_PAGE = 6;

    // Static function other mods can call to inject their buttons
    static void registerButton(CCMenuItemSpriteExtra* button) {
        if (button) {
            button->retain(); // Keep it alive
            registeredButtons.push_back(button);
        }
    }
};

std::vector<CCMenuItemSpriteExtra*> ToolloaderManager::registeredButtons = {};
int ToolloaderManager::currentPage = 0;

// Custom CCNode to hold and display the paginated toolbar
class PagingToolbar : public CCNode {
private:
    CCMenu* m_buttonMenu = nullptr;
    CCMenuItemSpriteExtra* m_prevBtn = nullptr;
    CCMenuItemSpriteExtra* m_nextBtn = nullptr;

public:
    bool init() {
        if (!CCNode::init()) return false;

        this->setContentSize({ 200.0f, 120.0f });

        // Main menu container for the 6 functional buttons
        m_buttonMenu = CCMenu::create();
        m_buttonMenu->setPosition({ 100.0f, 60.0f });
        this->addChild(m_buttonMenu);

        // Create Navigation Arrows using base game sprites
        auto prevSprite = CCSprite::createWithSpriteFrameName("GJ_arrow_01_001.png");
        prevSprite->setFlipX(true);
        prevSprite->setScale(0.6f);
        m_prevBtn = CCMenuItemSpriteExtra::create(
            prevSprite, this, menu_selector(PagingToolbar::onPrevPage)
        );
        m_prevBtn->setPosition({ -15.0f, 60.0f });

        auto nextSprite = CCSprite::createWithSpriteFrameName("GJ_arrow_01_001.png");
        nextSprite->setScale(0.6f);
        m_nextBtn = CCMenuItemSpriteExtra::create(
            nextSprite, this, menu_selector(PagingToolbar::onNextPage)
        );
        m_nextBtn->setPosition({ 215.0f, 60.0f });

        // Navigation menu container
        auto navMenu = CCMenu::create();
        navMenu->setPosition({ 0.0f, 0.0f });
        navMenu->addChild(m_prevBtn);
        navMenu->addChild(m_nextBtn);
        this->addChild(navMenu);

        updatePage();
        return true;
    }

    void updatePage() {
        m_buttonMenu->removeAllChildrenWithCleanup(false);

        int totalButtons = ToolloaderManager::registeredButtons.size();
        int startIndex = ToolloaderManager::currentPage * ToolloaderManager::BUTTONS_PER_PAGE;
        int endIndex = std::min(startIndex + ToolloaderManager::BUTTONS_PER_PAGE, totalButtons);

        // Visibility of navigation arrows
        m_prevBtn->setVisible(ToolloaderManager::currentPage > 0);
        m_nextBtn->setVisible(endIndex < totalButtons);

        // Create a 3x2 grid layout manually for clean positioning
        float startX = -60.0f;
        float startY = 30.0f;
        float spacingX = 60.0f;
        float spacingY = -60.0f;

        int count = 0;
        for (int i = startIndex; i < endIndex; ++i) {
            auto btn = ToolloaderManager::registeredButtons[i];
            
            int row = count / 3;
            int col = count % 3;

            btn->setPosition({ startX + (col * spacingX), startY + (row * spacingY) });
            m_buttonMenu->addChild(btn);
            count++;
        }
    }

    void onPrevPage(CCObject*) {
        if (ToolloaderManager::currentPage > 0) {
            ToolloaderManager::currentPage--;
            updatePage();
        }
    }

    void onNextPage(CCObject*) {
        int totalButtons = ToolloaderManager::registeredButtons.size();
        int maxPage = (totalButtons - 1) / ToolloaderManager::BUTTONS_PER_PAGE;
        if (ToolloaderManager::currentPage < maxPage) {
            ToolloaderManager::currentPage++;
            updatePage();
        }
    }

    static PagingToolbar* create() {
        auto ret = new PagingToolbar();
        if (ret && ret->init()) {
            ret->autorelease();
            return ret;
        }
        CC_SAFE_DELETE(ret);
        return nullptr;
    }
};

// Hook into MenuLayer to inject the toolloader UI and gather dummy buttons for testing
class $modify(MyMenuLayer, MenuLayer) {
    bool init() {
        if (!MenuLayer::init()) return false;

        // --- SIMULATED DETECTOR / MOD INJECTIONS ---
        if (ToolloaderManager::registeredButtons.empty()) {
            for (int i = 1; i <= 14; ++i) {
                auto label = CCLabelBMFont::create(std::to_string(i).c_str(), "bigFont.fnt");
                label->setScale(0.5f);
                
                auto btn = CCMenuItemSpriteExtra::create(
                    label, this, menu_selector(MyMenuLayer::onDummyClick)
                );
                btn->setTag(i);
                ToolloaderManager::registerButton(btn);
            }
        }
        // --------------------------------------------

        // Add the Paging Toolbar UI to the main menu screen
        auto toolbar = PagingToolbar::create();
        toolbar->setPosition({ 15.0f, 15.0f }); // Position it in the bottom left area
        this->addChild(toolbar);

        return true;
    }

    void onDummyClick(CCObject* sender) {
        auto btn = static_cast<CCMenuItemSpriteExtra*>(sender);
        FLAlertLayer::create("Toolloader", "Clicked Mod Button: " + std::to_string(btn->getTag()), "OK")->show();
    }
};
