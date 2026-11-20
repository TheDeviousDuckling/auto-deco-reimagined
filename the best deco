#include <Geode/Geode.hpp>
#include <Geode/modify/EditorUI.hpp>
#include <random>
#include <vector>
#include <cmath>
#include <algorithm>

using namespace geode::prelude;

struct Theme {
	const char* name;
	std::vector<int> topIDs;
	std::vector<int> bottomIDs;
	std::vector<int> scatterIDs;
	float minScale, maxScale;
	float perBlock;
	float rotJitter;
};

static const std::vector<Theme>& themes() {
	static const std::vector<Theme> t = {
		{ "Acheron",    {39, 8},  {39},    {1, 39}, 0.35f, 0.9f, 1.5f, 45.f  },
		{ "Space",      {},       {},      {1, 40}, 0.10f, 0.35f, 2.5f, 180.f },
		{ "Grasslands", {39},     {},      {},      0.25f, 0.6f, 3.0f, 12.f  },
		{ "Volcano",    {8, 39},  {8, 39}, {1},     0.4f,  1.0f, 1.2f, 20.f  },
		{ "Ocean",      {},       {39},    {1, 40}, 0.12f, 0.4f, 2.0f, 180.f },
		{ "Cyber",      {40},     {40},    {1},     0.3f,  0.7f, 1.5f, 0.f   },
		{ "Crystal",    {8, 39},  {8},     {39},    0.3f,  0.8f, 1.3f, 35.f  },
	};
	return t;
}

static std::mt19937& rng() {
	static std::mt19937 r{std::random_device{}()};
	return r;
}
static float rf(float a, float b) {
	if (a >= b) return a;
	return std::uniform_real_distribution<float>(a, b)(rng());
}
static int pick(const std::vector<int>& v) {
	return v[std::uniform_int_distribution<size_t>(0, v.size() - 1)(rng())];
}

static void spawn(LevelEditorLayer* lel, int id, CCPoint pos, float scale, float rot) {
	GameObject* obj = lel->createObject(id, pos, true);
	if (!obj) return;
	obj->updateCustomScaleX(scale);
	obj->updateCustomScaleY(scale);
	obj->setRotation(rot);
}

class $modify(AutoDecoUI, EditorUI) {
	struct Fields {
		size_t theme = 0;
		ButtonSprite* themeSpr = nullptr;
	};

	bool init(LevelEditorLayer* lel) {
		if (!EditorUI::init(lel)) return false;

		auto win = CCDirector::get()->getWinSize();
		auto menu = CCMenu::create();
		menu->setPosition({win.width - 70.f, 110.f});

		auto themeSpr = ButtonSprite::create(themes()[0].name);
		m_fields->themeSpr = themeSpr;
		auto themeBtn = CCMenuItemSpriteExtra::create(themeSpr, this, menu_selector(AutoDecoUI::onTheme));
		themeBtn->setPosition({0.f, 22.f});

		auto goSpr = ButtonSprite::create("Decorate");
		auto goBtn = CCMenuItemSpriteExtra::create(goSpr, this, menu_selector(AutoDecoUI::onDecorate));
		goBtn->setPosition({0.f, -22.f});

		menu->addChild(themeBtn);
		menu->addChild(goBtn);
		this->addChild(menu, 100);
		return true;
	}

	void onTheme(CCObject*) {
		m_fields->theme = (m_fields->theme + 1) % themes().size();
		m_fields->themeSpr->setString(themes()[m_fields->theme].name);
	}

	void onDecorate(CCObject*) {
		std::vector<GameObject*> bases;

		CCArray* sel = this->getSelectedObjectsOfCCArray();
		if (sel) {
			for (unsigned int i = 0; i < sel->count(); i++) {
				bases.push_back(static_cast<GameObject*>(sel->objectAtIndex(i)));
			}
		}
		if (bases.empty()) {
			FLAlertLayer::create("Auto Decorator", "Select some blocks first.", "OK")->show();
			return;
		}

		float density = (float)Mod::get()->getSettingValue<double>("density");
		const Theme& th = themes()[m_fields->theme];

		for (GameObject* base : bases) {
			CCPoint p = base->getPosition();
			CCSize s = base->getScaledContentSize();
			float halfW = s.width / 2.f;
			float halfH = s.height / 2.f;
			int count = std::max(1, (int)std::round(th.perBlock * density));

			for (int i = 0; i < count; i++) {
				float sc = rf(th.minScale, th.maxScale);
				float x = p.x + rf(-halfW, halfW);

				if (!th.topIDs.empty()) {
					spawn(m_editorLayer, pick(th.topIDs), {x, p.y + halfH + 15.f * sc}, sc, rf(-th.rotJitter, th.rotJitter));
				}
				if (!th.bottomIDs.empty()) {
					spawn(m_editorLayer, pick(th.bottomIDs), {x, p.y - halfH - 15.f * sc}, sc, 180.f + rf(-th.rotJitter, th.rotJitter));
				}
				if (!th.scatterIDs.empty()) {
					spawn(m_editorLayer, pick(th.scatterIDs), {p.x + rf(-halfW - 45.f, halfW + 45.f), p.y + rf(-halfH - 60.f, halfH + 60.f)}, sc, rf(0.f, 360.f));
				}
			}
		}
	}
};
