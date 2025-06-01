#include <Security/Api/api.hpp>
#include "Gui.hpp"

#include <Includes/CustomWidgets/Custom.hpp>
#include <Includes/CustomWidgets/WaterMarks.hpp>
#include <Includes/CustomWidgets/Notify.hpp>

#include <Gui/Pages/Combat.hpp>
#include <Gui/Pages/Visuals.hpp>
#include <Gui/Pages/Local.hpp>
#include <Gui/Pages/Exploits.hpp>
#include <Gui/Pages/World.hpp>
#include <Gui/Pages/Settings.hpp>
#include <Gui/Pages/Login.hpp>
#include <Core/Features/Exploits/Exploits.hpp>
#include <Includes/CustomWidgets/Notify.hpp>
#include <Core/Features/Esp.hpp>

#include "../Includes/skstr.hpp"
#include "Overlay/Overlay.hpp"
#include <Core/Features/Exploits/HandlingEditor.hpp>
#include <Core/Features/Exploits/GiveWeapon.hpp>  

int PauseLoop;
inline std::mutex DrawMtx;

float content_animation;
float accent_colour[4];

const char* bones[]{ "Head", "Chest", "Leg" };
const char* healthBarTypes[]{ "Top", "Right", "Bottom", "Left" };
const char* textTypes[]{ "Top", "Bottom" };

void legit_tab() {
	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Override config"), ImVec2(320, 150));
	{
		static bool waitingforkeyAim = false;

		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("Aim Assist Key"));
		ImGui::SetCursorPosX(20);
		ImGui::Hotkey(&g_Config.Aimbot->KeyBind, ImVec2(320 - 38, 25), &waitingforkeyAim, "- Aim");

		ImGui::Spacing();
		ImGui::Combo(skCrypt("HitBox"), &g_Config.Aimbot->HitBox, bones, IM_ARRAYSIZE(bones));
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(169, 227));
	ImGui::MenuChild(skCrypt("General"), ImVec2(320, 292));
	{
		ImGui::Spacing();
		ImGui::Checkbox(skCrypt("Enable Aim Assist"), &g_Config.Aimbot->Enabled);

		ImGui::Checkbox(skCrypt("Show Fov"), &g_Config.Aimbot->ShowFov);
		if (g_Config.Aimbot->ShowFov) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 38);
			if (ImGui::ColorButton(skCrypt("Fov Color"), g_Config.Aimbot->FovColor)) {
				ImGui::OpenPopup(skCrypt("FovColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("FovColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.Aimbot->FovColor,
					ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar |
					ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Accuracy"), ImVec2(320, 286));
	{
		ImGui::Spacing();
		ImGui::SliderInt(xorstr("Field of View"), &g_Config.Aimbot->FOV, 0, 1000, xorstr("%d"), 0);
		ImGui::SliderInt(xorstr("Smooth"), &g_Config.Aimbot->AimbotSpeed, 0, 100, xorstr("%d"), 0);
		ImGui::SliderInt(xorstr("Max Distance"), &g_Config.Aimbot->MaxDistance, 0, 1000, xorstr("%dm"), 0);
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 363));
	ImGui::MenuChild(skCrypt("Misc"), ImVec2(320, 157));
	{
		ImGui::Spacing();
		ImGui::Checkbox(xorstr("Only Visible"), &g_Config.Aimbot->OnlyVisible);
		ImGui::Checkbox(xorstr("Ignore NPCs"), &g_Config.Aimbot->IgnoreNPCs);
	}
	ImGui::EndChild();
}

void silent_tab() {
	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Override config"), ImVec2(320, 80));
	{
		static bool waitingforkeyRage = false;

		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("Silent Aim Key"));
		ImGui::SetCursorPosX(20);
		ImGui::Hotkey(&g_Config.SilentAim->KeyBind, ImVec2(320 - 38, 25), &waitingforkeyRage, "- Silent");

		ImGui::Spacing();
		ImGui::Combo(skCrypt("HitBox"), &g_Config.SilentAim->Hitbox, bones, IM_ARRAYSIZE(bones));
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(169, 156));
	ImGui::MenuChild(skCrypt("General"), ImVec2(320, 363));
	{
		ImGui::Spacing();
		ImGui::Checkbox(xorstr("Toggle"), &g_Config.SilentAim->Enabled);

		ImGui::Checkbox(skCrypt("Show Fov"), &g_Config.SilentAim->ShowFov);
		if (g_Config.SilentAim->ShowFov) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 38);
			if (ImGui::ColorButton(skCrypt("Fov Color"), g_Config.SilentAim->FovColor)) {
				ImGui::OpenPopup(skCrypt("FovColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("FovColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.SilentAim->FovColor, ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar | ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Accuracy"), ImVec2(320, 286));
	{
		ImGui::Spacing();

		ImGui::SliderInt(xorstr("Field of View"), &g_Config.SilentAim->FOV, 0, 1000, xorstr("%d"), 0);
		ImGui::SliderInt(xorstr("Miss Chance"), &g_Config.SilentAim->MissChance, 0, 100, xorstr("%dx"), 0);
		ImGui::SliderInt(xorstr("Max Distance"), &g_Config.SilentAim->MaxDistance, 0, 1000, xorstr("%dm"), 0);
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 363));
	ImGui::MenuChild(skCrypt("Misc"), ImVec2(320, 157));
	{
		ImGui::Spacing();
		ImGui::Checkbox(xorstr("Magic Bullets"), &g_Config.SilentAim->MagicBullets);
		ImGui::Checkbox(xorstr("Only Visible"), &g_Config.SilentAim->OnlyVisible);
		ImGui::Checkbox(xorstr("Ignore NPCs"), &g_Config.SilentAim->IgnoreNPCs);
	}
	ImGui::EndChild();
}

void Particles()
{
	if (!Gui::cOverlay.particles)
		return;

	ImVec2 screen_size = { (float)GetSystemMetrics(SM_CXSCREEN), (float)GetSystemMetrics(SM_CYSCREEN) };

	static ImVec2 particle_pos[300];
	static ImVec2 particle_target_pos[300];
	static float particle_speed[300];
	static float particle_radius[300];

	for (int i = 1; i < Gui::cOverlay.particleCount; i++)
	{
		if (particle_pos[i].x == 0 || particle_pos[i].y == 0)
		{
			particle_pos[i].x = rand() % (int)screen_size.x + 1;
			particle_pos[i].y = 15.f;
			particle_speed[i] = Gui::cOverlay.minSpeed + static_cast<float>(rand()) / (static_cast<float>(RAND_MAX / (Gui::cOverlay.maxSpeed - Gui::cOverlay.minSpeed)));
			particle_radius[i] = Gui::cOverlay.minRadius + static_cast<float>(rand()) / (static_cast<float>(RAND_MAX / (Gui::cOverlay.maxRadius - Gui::cOverlay.minRadius)));

			particle_target_pos[i].x = rand() % (int)screen_size.x;
			particle_target_pos[i].y = screen_size.y * 2;
		}

		particle_pos[i] = ImLerp(particle_pos[i], particle_target_pos[i], ImGui::GetIO().DeltaTime * (particle_speed[i] / 60));

		if (particle_pos[i].y > screen_size.y)
		{
			particle_pos[i].x = 0;
			particle_pos[i].y = 0;
		}

		ImColor particlesColorWithAlpha = ImColor(Gui::cOverlay.particlesColor.Value.x, Gui::cOverlay.particlesColor.Value.y, Gui::cOverlay.particlesColor.Value.z, Gui::cOverlay.particlesColor.Value.w * ImGui::GetStyle().Alpha);
		ImGui::GetWindowDrawList()->AddCircleFilled(particle_pos[i], particle_radius[i], particlesColorWithAlpha);
	}
}

void trigger_tab() {
	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Override config"), ImVec2(320, 80));
	{
		static bool waitingforkeyTriggerEnable = false;
		static bool waitingforkeyTrigger = false;

		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("Trigger Key"));
		ImGui::SetCursorPosX(20);
		ImGui::Hotkey(&g_Config.TriggerBot->KeyBind, ImVec2(320 - 38, 25), &waitingforkeyTrigger, skCrypt("- Trigger"));

	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(169, 156));
	ImGui::MenuChild(skCrypt("General"), ImVec2(320, 363));
	{
		ImGui::Spacing();
		ImGui::Checkbox(xorstr("Toggle"), &g_Config.TriggerBot->Enabled);

		ImGui::Checkbox(xorstr("Smart Triggerbot"), &g_Config.TriggerBot->SmartTrigger);
		if (!g_Config.TriggerBot->SmartTrigger) {
			ImGui::Checkbox(xorstr("Show Fov"), &g_Config.TriggerBot->ShowFov);
		}
		else
		{
			g_Config.TriggerBot->ShowFov = false;
		}

	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Accuracy"), ImVec2(320, 286));
	{
		ImGui::Spacing();

		if (!g_Config.TriggerBot->SmartTrigger) {
			ImGui::SliderInt(xorstr("Field of View"), &g_Config.TriggerBot->FOV, 0, 400, xorstr("%d"), 0);
			ImGui::SliderInt(xorstr("Max Distance"), &g_Config.TriggerBot->MaxDistance, 0, 1000, xorstr("%dm"), 0);
		}

		ImGui::SliderInt(xorstr("Delay"), &g_Config.TriggerBot->Delay, 0, 10, xorstr("%d"), 0);

	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 363));
	ImGui::MenuChild(skCrypt("Misc"), ImVec2(320, 157));
	{
		ImGui::Spacing();
		ImGui::Checkbox(xorstr("Only Visible"), &g_Config.TriggerBot->OnlyVisible);
		ImGui::Checkbox(xorstr("Ignore NPCs"), &g_Config.TriggerBot->IgnoreNPCs);
	}
	ImGui::EndChild();
}

void players_tab() {
	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Override config"), ImVec2(320, 80));
	{
		static bool waitingforkeyEspEnable = false;

		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("Esp Enable Key"));
		ImGui::SetCursorPosX(20);
		ImGui::Hotkey(&g_Config.ESP->KeyBind, ImVec2(320 - 38, 25), &waitingforkeyEspEnable, "- Enable");
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(169, 156));
	ImGui::MenuChild(skCrypt("General"), ImVec2(320, 363));
	{
		ImGui::Spacing();
		ImGui::Checkbox(skCrypt("Enable Esp Player"), &g_Config.ESP->Enabled);
		ImGui::Checkbox(skCrypt("Box"), &g_Config.ESP->Box);
		if (g_Config.ESP->Box) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 31);
			if (ImGui::ColorButton(skCrypt("Box Color"), g_Config.ESP->BoxCol)) {
				ImGui::OpenPopup(skCrypt("BoxColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("BoxColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.ESP->BoxCol, ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar | ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}

		}

		ImGui::Checkbox(skCrypt("Skeleton"), &g_Config.ESP->Skeleton);
		if (g_Config.ESP->Skeleton) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 31);
			if (ImGui::ColorButton(skCrypt("Skeleton Color"), g_Config.ESP->SkeletonCol)) {
				ImGui::OpenPopup(skCrypt("SkeletonColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("SkeletonColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.ESP->SkeletonCol, ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar | ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}

		}
		ImGui::Checkbox(skCrypt("Name"), &g_Config.ESP->UserNames);

		ImGui::Checkbox(skCrypt("Health Bar"), &g_Config.ESP->HealthBar);

		ImGui::Checkbox(skCrypt("Armor Bar"), &g_Config.ESP->ArmorBar);
		ImGui::Checkbox(skCrypt("Distance"), &g_Config.ESP->DistanceFromMe);
		if (g_Config.ESP->DistanceFromMe) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 31);
			if (ImGui::ColorButton(skCrypt("Distance Color"), g_Config.ESP->DistanceCol)) {
				ImGui::OpenPopup(skCrypt("DistanceColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("DistanceColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.ESP->DistanceCol, ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar | ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}

		}
		ImGui::Checkbox(skCrypt("Weapon"), &g_Config.ESP->WeaponName);
		if (g_Config.ESP->WeaponName) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 31);
			if (ImGui::ColorButton(skCrypt("Weapon Color"), g_Config.ESP->WeaponNameCol)) {
				ImGui::OpenPopup(skCrypt("WeaponColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("WeaponColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.ESP->WeaponNameCol, ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar | ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}

		}

		ImGui::Checkbox(skCrypt("Lines"), &g_Config.ESP->SnapLines);
		if (g_Config.ESP->SnapLines) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 31);
			if (ImGui::ColorButton(skCrypt("Lines Color"), g_Config.ESP->SnapLinesCol)) {
				ImGui::OpenPopup(skCrypt("LinesColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("LinesColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.ESP->SnapLinesCol, ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar | ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}
		ImGui::Spacing();
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Accuracy"), ImVec2(320, 286));
	{
		ImGui::Spacing();
		ImGui::SliderInt(skCrypt("Distance"), &g_Config.ESP->MaxDistance, 0, 500);
		if (g_Config.ESP->HealthBar) {
			ImGui::Combo(skCrypt("Health Bar Type"), &g_Config.ESP->HealthBarState, healthBarTypes, IM_ARRAYSIZE(healthBarTypes), ImGuiComboFlags_NoArrowButton);
		}
		if (g_Config.ESP->ArmorBar) {
			ImGui::Combo(skCrypt("Armor Bar Type"), &g_Config.ESP->ArmorBarState, healthBarTypes, IM_ARRAYSIZE(healthBarTypes), ImGuiComboFlags_NoArrowButton);
		}

		if (g_Config.ESP->UserNames) {
			ImGui::Combo(skCrypt("Username State"), &g_Config.ESP->UserNamesState, textTypes, IM_ARRAYSIZE(textTypes), ImGuiComboFlags_NoArrowButton);
		}
		if (g_Config.ESP->DistanceFromMe) {
			ImGui::Combo(skCrypt("Distance Type"), &g_Config.ESP->DistanceFromMeState, textTypes, IM_ARRAYSIZE(textTypes), ImGuiComboFlags_NoArrowButton);
		}
		if (g_Config.ESP->WeaponName) {
			ImGui::Combo(skCrypt("Weapon Type"), &g_Config.ESP->WeaponNameState, textTypes, IM_ARRAYSIZE(textTypes), ImGuiComboFlags_NoArrowButton);
		}

		ImGui::Spacing();
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 363));
	ImGui::MenuChild(skCrypt("Misc"), ImVec2(320, 157));
	{
		ImGui::Spacing();
		ImGui::Checkbox(skCrypt("Ignore Ped"), &g_Config.ESP->IgnoreNPCs);
		ImGui::Checkbox(skCrypt("Ignore Dead"), &g_Config.ESP->IgnoreDead);
		ImGui::Checkbox(skCrypt("Show Local Player"), &g_Config.ESP->ShowLocalPlayer);
	}
	ImGui::EndChild();
}

void vehicles_tab() {
	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Override config"), ImVec2(320, 80));
	{
		ImGui::Spacing();
		ImGui::Checkbox(skCrypt("Toggle"), &g_Config.VehicleESP->Enabled);
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(169, 156));
	ImGui::MenuChild(skCrypt("General"), ImVec2(320, 363));
	{
		bool InVehicle = Core::SDK::Pointers::pLocalPlayer->InVehicle();
		auto CurrentVehicle = Core::SDK::Pointers::pLocalPlayer->GetLastVehicle();

		ImGui::Spacing();

		ImGui::Checkbox(skCrypt("Vehicle Names"), &g_Config.VehicleESP->VehName);
		ImGui::Checkbox(skCrypt("Locked/Unlocked"), &g_Config.VehicleESP->ShowLockUnlock);

		ImGui::Checkbox(skCrypt("SnapLines"), &g_Config.VehicleESP->SnapLines);
		if (g_Config.VehicleESP->SnapLines) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 31);
			if (ImGui::ColorButton(skCrypt("SnapLines Color"), g_Config.VehicleESP->SnapLinesCol)) {
				ImGui::OpenPopup(skCrypt("SnapLinesColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("SnapLinesColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&g_Config.VehicleESP->SnapLinesCol,
					ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar |
					ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}

		ImGui::Checkbox(skCrypt("Distance"), &g_Config.VehicleESP->DistanceFromMe);

		if (ImGui::Checkbox(skCrypt("Vehicle GodMode"), &g_Config.Player->VehicleGodMode)) {
			if (!InVehicle) {
				g_Config.Player->VehicleGodMode = false;
				std::thread VehicleAlert([]() { NotifyManager::Send(skCrypt("You must be in a car to use this function").decrypt(), 4000); });
				VehicleAlert.detach();
			}
			else {
				CurrentVehicle->SetGodMode(g_Config.Player->VehicleGodMode);
			}
		}

		static bool Locked = CurrentVehicle->IsLocked();
		if (ImGui::Checkbox(skCrypt("Doors Locked"), &Locked)) {
			CurrentVehicle->DoorState(!Locked);
		}

		if (ImGui::Checkbox(skCrypt("SeatBelt"), &g_Config.Player->SeatBelt)) {
			if (!InVehicle) {
				g_Config.Player->SeatBelt = false;
				std::thread VehicleAlert([]() { NotifyManager::Send(skCrypt("You must be in a car to use this function").decrypt(), 4000); });
				VehicleAlert.detach();
			}
			else {
				std::thread SeatBelt([]() {
					Core::SDK::Pointers::pLocalPlayer->SeatBealt(g_Config.Player->SeatBelt);
					});
				SeatBelt.detach();
			}
		}	
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Settings"), ImVec2(320, 481)); // Increased height to accommodate new content
	{
		bool InVehicle = Core::SDK::Pointers::pLocalPlayer->InVehicle();

		ImGui::Spacing();
		ImGui::SliderInt(skCrypt("Max Distance"), &g_Config.VehicleESP->MaxDistance, 0, 1000, skCrypt("%dm"), 0);

		ImGui::SetCursorPosX(20);
		if (ImGui::Button(skCrypt("Repair Vehicle"), ImVec2(320 - 38, 30))) {
			Core::SDK::Pointers::pLocalPlayer->GetLastVehicle()->Fix();
		}

		ImGui::Spacing();

		if (InVehicle)
		{
			ImGui::Text(skCrypt("Handling Editor"));

			if (ImGui::SliderFloat(skCrypt("Acceleration"), &Features::Exploits::g_HandlingEditor.fAcceleration, 0.0f, 400.f, skCrypt("%1.1f")))
				Features::Exploits::g_HandlingEditor.ApplyHandlingValues();

			if (ImGui::SliderFloat(skCrypt("Brake Force"), &Features::Exploits::g_HandlingEditor.fBrakeForce, 0.0f, 100.f, skCrypt("%1.1f")))
				Features::Exploits::g_HandlingEditor.ApplyHandlingValues();

			if (ImGui::SliderFloat(skCrypt("Traction Curve Min"), &Features::Exploits::g_HandlingEditor.fTractionCurveMin, 0.0f, 100.f, skCrypt("%1.1f")))
				Features::Exploits::g_HandlingEditor.ApplyHandlingValues();
		}
		else if (g_Config.Player->HandlingEditor && !InVehicle) {
			ImGui::Text(skCrypt("Handling Editor"));
			ImGui::Text(skCrypt("You must be in a vehicle"));
		}
	}
	ImGui::EndChild();
}

void world_tab() {
	static int selectedPlayer = -1;
	static int selectedVehicle = -1;

	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Lists"), ImVec2(320, 481));
	{
		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("Players"));
		ImGui::SetCursorPosX(20);
		std::vector<const char*> playerNames;
		for (int i = 0; i < Core::SDK::Game::EntityList.size(); i++) {
			if (Core::SDK::Game::EntityList[i].Ped == Core::SDK::Pointers::pLocalPlayer)
				continue;
			if (Core::SDK::Game::EntityList[i].PedType != 2)
				continue;
			playerNames.push_back(Core::SDK::Game::EntityList[i].NetworkInfo.UserName.c_str());
		}
		ImGui::ListBox(skCrypt("##Players"), &selectedPlayer, playerNames.data(), playerNames.size(), 8);

		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("Vehicles"));
		ImGui::SetCursorPosX(20);
		std::vector<std::string> vehicleNames;
		{
			std::lock_guard<std::mutex> lock(Core::Threads::g_VehicleList.vehicleListMutex);
			vehicleNames.resize(Core::SDK::Game::VehicleList.size()); // Pre-allocate to avoid reallocation
			for (int i = 0; i < Core::SDK::Game::VehicleList.size(); i++) {
				std::string vehicleName = Core::SDK::Game::VehicleList[i].Name.empty() ||
					Core::SDK::Game::VehicleList[i].Name == "" ?
					"Unknown Vehicle" : Core::SDK::Game::VehicleList[i].Name;
				vehicleNames[i] = vehicleName + " (" +
					std::to_string((int)Core::SDK::Game::VehicleList[i].Dist) + "m)";
			}
		}
		std::vector<const char*> vehicleNamesCStr;
		vehicleNamesCStr.reserve(vehicleNames.size()); // Reserve space to avoid reallocation
		for (const auto& name : vehicleNames) {
			vehicleNamesCStr.push_back(name.c_str());
		}
		ImGui::ListBox(skCrypt("##Vehicles"), &selectedVehicle, vehicleNamesCStr.data(), vehicleNamesCStr.size(), 8);
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Actions"), ImVec2(320, 481));
	{
		ImGui::Spacing();

		ImGui::Text(skCrypt("Player Info & Actions"));
		if (selectedPlayer == -1 || selectedPlayer >= Core::SDK::Game::EntityList.size()) {
			ImGui::Text(skCrypt("Select a player"));
		}
		else {
			auto& selectedPed = Core::SDK::Game::EntityList[selectedPlayer];

			ImGui::Text(skCrypt("Name: %s"), selectedPed.NetworkInfo.UserName.c_str());
			ImGui::Text(skCrypt("Id: %d"), selectedPed.Id);
			ImGui::Text(skCrypt("Distance: %.2f"), selectedPed.Distance);
			ImGui::Text(skCrypt("Weapon: %s"), selectedPed.WeaponName.c_str());
			ImGui::Text(skCrypt("Friend: %s"), selectedPed.IsFriend ? "True" : "False");

			ImGui::Spacing();

			bool isFriend = selectedPed.IsFriend;
			if (ImGui::Checkbox(skCrypt("Friend"), &isFriend)) {
				Core::SDK::Game::FriendMap[selectedPed.Ped] = isFriend;
			}

			ImGui::SetCursorPosX(20);
			if (ImGui::Button(skCrypt("Teleport to Player"), ImVec2(320 - 38, 30))) {
				Core::SDK::Pointers::pLocalPlayer->SetPos(selectedPed.Pos);
			}

			ImGui::SetCursorPosX(20);
			if (ImGui::Button(skCrypt("Copy Clothes"), ImVec2(320 - 38, 30))) {
				if (!selectedPed.Ped) return;

				uintptr_t pedAddress = reinterpret_cast<uintptr_t>(selectedPed.Ped);
				uint64_t drawhandler = Mem.Read<uint64_t>(pedAddress + 0x48);

				uint64_t shoes = Mem.Read<uint64_t>(drawhandler + 0xF0);
				uint64_t leg = Mem.Read<uint64_t>(drawhandler + 0xF8);
				uint64_t body = Mem.Read<uint64_t>(drawhandler + 0x100);
				uint64_t tshirt = Mem.Read<uint64_t>(drawhandler + 0x108);
				uint64_t mask = Mem.Read<uint64_t>(drawhandler + 0x110);
				uint64_t hand = Mem.Read<uint64_t>(drawhandler + 0xF4);
				uint64_t bag = Mem.Read<uint64_t>(drawhandler + 0xFC);
				uint64_t armor = Mem.Read<uint64_t>(drawhandler + 0x10C);

				uintptr_t localPlayerAddr = reinterpret_cast<uintptr_t>(Core::SDK::Pointers::pLocalPlayer);
				uint64_t local_drawhandler = Mem.Read<uint64_t>(localPlayerAddr + 0x48);

				Mem.Write<uint64_t>(local_drawhandler + 0xF0, shoes);
				Mem.Write<uint64_t>(local_drawhandler + 0xF8, leg);
				Mem.Write<uint64_t>(local_drawhandler + 0x100, body);
				Mem.Write<uint64_t>(local_drawhandler + 0x108, tshirt);
				Mem.Write<uint64_t>(local_drawhandler + 0x110, mask);
				Mem.Write<uint64_t>(local_drawhandler + 0xF4, hand);
				Mem.Write<uint64_t>(local_drawhandler + 0xFC, bag);
				Mem.Write<uint64_t>(local_drawhandler + 0x10C, armor);

				Mem.Write<uint64_t>(localPlayerAddr + 0xAC, 0);
			}
		}

		ImGui::Spacing();

		ImGui::Text(skCrypt("Vehicle Info & Actions"));
		if (selectedVehicle == -1 || selectedVehicle >= Core::SDK::Game::VehicleList.size()) {
			ImGui::Text(skCrypt("Select a vehicle"));
		}
		else {
			std::lock_guard<std::mutex> lock(Core::Threads::g_VehicleList.vehicleListMutex);

			auto& selectedVeh = Core::SDK::Game::VehicleList[selectedVehicle];

			int pedId = reinterpret_cast<int>(selectedVeh.Driver);

			std::string vehicleName = selectedVeh.Name.empty() || selectedVeh.Name == "" ?
				"Unknown Vehicle" : selectedVeh.Name;
			ImGui::Text(skCrypt("Name: %s"), vehicleName.c_str());
			ImGui::Text(skCrypt("Distance: %dm"), (int)selectedVeh.Dist);
			ImGui::Text(skCrypt("Driver: %s"),
				selectedVeh.Driver == 0 ? "Unknown" :
				selectedVeh.Driver->GetPedType() != 2 ? "NPC" :
				Core::SDK::Game::GetPedName(pedId, selectedVeh.Driver).c_str());

			ImGui::Text(skCrypt("Doors Locked: %s"), selectedVeh.IsLocked ? "Yes" : "No");

			ImGui::Spacing();

			ImGui::SetCursorPosX(20);
			if (selectedVeh.Pointer->IsLocked()) {
				if (ImGui::Button(skCrypt("Unlock Vehicle"), ImVec2(320 - 38, 30))) {
					selectedVeh.Pointer->DoorState(true);
				}
			}
			else {
				if (ImGui::Button(skCrypt("Lock Vehicle"), ImVec2(320 - 38, 30))) {
					selectedVeh.Pointer->DoorState(false);
				}
			}

			ImGui::SetCursorPosX(20);
			if (ImGui::Button(skCrypt("Teleport to Vehicle"), ImVec2(320 - 38, 30))) {
				Core::SDK::Pointers::pLocalPlayer->SetPos(selectedVeh.Pointer->GetPos());
			}
		}
	}
	ImGui::EndChild();
}

void exploits_tab() {
	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("General"), ImVec2(320, 481));
	{
		ImGui::Spacing();

		if (ImGui::Checkbox(skCrypt("GodMode"), &g_Config.Player->EnableGodMode)) {
			Core::SDK::Pointers::pLocalPlayer->SetGodMode(g_Config.Player->EnableGodMode);
		}

		if (ImGui::Checkbox(skCrypt("NoClip"), &g_Config.Player->NoClipEnabled)) {
			Core::SDK::Pointers::pLocalPlayer->FreezePed(g_Config.Player->NoClipEnabled);
		}

		ImGui::Checkbox(skCrypt("Fast Run"), &g_Config.Player->FastRun);

		if (ImGui::Checkbox(skCrypt("Shrink"), &g_Config.Player->ShrinkEnabled)) {
			Core::SDK::Pointers::pLocalPlayer->SetConfigFlag(ePedConfigFlag::Shrink, g_Config.Player->ShrinkEnabled);
		}

		static bool AntiHSActive = false;

		if (ImGui::Checkbox("Anti Headshot", &g_Config.Player->AntiHSEnabled)) {
			if (g_Config.Player->AntiHSEnabled && !AntiHSActive) {
				AntiHSActive = true;
				std::thread([] {
					while (AntiHSActive) {
						if (Core::SDK::Pointers::pLocalPlayer)
							Core::SDK::Pointers::pLocalPlayer->SetHealth(10000);

						std::this_thread::sleep_for(std::chrono::seconds(15));
					}
					}).detach();
			}
			else if (!g_Config.Player->AntiHSEnabled && AntiHSActive) {
				AntiHSActive = false;
			}
		}



		//

		if (ImGui::Checkbox(skCrypt("Beast Jump"), &g_Config.Player->BeastJumpEnabled)) {}

		if (g_Config.Player->BeastJumpEnabled && Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->BeastJump(true);
		}
		else if (Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->BeastJump(false);
		}

		//

		if (ImGui::Checkbox(skCrypt("Super Jump"), &g_Config.Player->SuperJumpEnabled)) {}

		if (g_Config.Player->SuperJumpEnabled && Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->SuperJump(true);
		}
		else if (Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->SuperJump(false);
		}

		//

		if (ImGui::Checkbox(skCrypt("Explosive Fist"), &g_Config.Player->ExplosiveFistEnabled)) {}

		if (g_Config.Player->ExplosiveFistEnabled && Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->ExplosiveFist(true);
		}
		else if (Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->ExplosiveFist(false);
		}

		//

		if (ImGui::Checkbox(skCrypt("Super Fist"), &g_Config.Player->SuperFistEnabled)) {}

		if (g_Config.Player->SuperFistEnabled && Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->SuperFistOn(true);
		}
		else if (Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->SuperFistOff(true);
		}

		//

		if (ImGui::Checkbox(skCrypt("No RagDoll"), &g_Config.Player->NoRagDollEnabled)) {
			Core::SDK::Pointers::pLocalPlayer->NoRagDoll(g_Config.Player->NoRagDollEnabled);
		}

		if (ImGui::Checkbox(xorstr("Steal Car"), &g_Config.Player->StealCarEnabled))
		{
			if (g_Config.Player->StealCarEnabled)
			{
				Core::SDK::Pointers::pLocalPlayer->SetConfigFlag(ePedConfigFlag::NotAllowedToJackAnyPlayers, false);
				Core::SDK::Pointers::pLocalPlayer->SetConfigFlag(ePedConfigFlag::PlayerCanJackFriendlyPlayers, true);
				Core::SDK::Pointers::pLocalPlayer->SetConfigFlag(ePedConfigFlag::WillJackAnyPlayer, true);
			}
			else
			{
				Core::SDK::Pointers::pLocalPlayer->SetConfigFlag(ePedConfigFlag::NotAllowedToJackAnyPlayers, true);
				Core::SDK::Pointers::pLocalPlayer->SetConfigFlag(ePedConfigFlag::PlayerCanJackFriendlyPlayers, false);
				Core::SDK::Pointers::pLocalPlayer->SetConfigFlag(ePedConfigFlag::WillJackAnyPlayer, false);
			}
		}

		if (ImGui::Checkbox(skCrypt("Infinite CombatRoll"), &g_Config.Player->InfiniteCombatRoll)) {
			std::thread InfiniteCombatRoll([]() { Core::SDK::Pointers::pLocalPlayer->SetInfCombatRoll(g_Config.Player->InfiniteCombatRoll); });
			InfiniteCombatRoll.detach();
		}

		if (ImGui::Checkbox(skCrypt("Infinite Stamina"), &g_Config.Player->InfiniteStamina)) {
			Core::SDK::Pointers::pLocalPlayer->SetInfStamina(g_Config.Player->InfiniteStamina);
		}

		if (ImGui::Checkbox(skCrypt("Force Weapon Wheel"), &g_Config.Player->ForceWeaponWheel)) {
			std::thread ForceWeaponWheel([]() { Core::SDK::Pointers::pLocalPlayer->ForceWeaponWheel(g_Config.Player->ForceWeaponWheel); });
			ForceWeaponWheel.detach();
		}

		if (ImGui::Checkbox(skCrypt("No Recoil"), &g_Config.Player->NoRecoilEnabled)) {
			if (g_Config.Player->NoRecoilEnabled) {
				Core::SDK::Pointers::pLocalPlayer->GetWeaponManager()->SetRecoil(0.0f);
			}
			else {
				Core::SDK::Pointers::pLocalPlayer->GetWeaponManager()->SetRecoil(g_Config.Player->RecoilValue);
			}
		}

		if (ImGui::Checkbox(xorstr("No Reload"), &g_Config.Player->NoReloadEnabled)) {
			if (g_Config.Player->NoReloadEnabled) {
				Mem.PatchFunc(g_Offsets.m_InfiniteAmmo0, 3);
			}
			else {
				Mem.WriteBytes(g_Offsets.m_InfiniteAmmo0, { 0x41, 0x2B, 0xC9, 0x3B, 0xC8, 0x0F, 0x4D, 0xC8 });
			}
		}

		if (ImGui::Checkbox(skCrypt("No Spread"), &g_Config.Player->NoSpreadEnabled)) {
			if (g_Config.Player->NoSpreadEnabled) {
				Core::SDK::Pointers::pLocalPlayer->GetWeaponManager()->SetSpread(0.0f);
			}
			else {
				Core::SDK::Pointers::pLocalPlayer->GetWeaponManager()->SetSpread(g_Config.Player->SpreadValue);
			}
		}

		ImGui::Checkbox(skCrypt("Bullet Damage"), &g_Config.Player->DamageBoost);

		if (ImGui::Checkbox(skCrypt("Explosive Ammo"), &g_Config.Player->ExplosiveAmmoEnabled)) {}

		if (g_Config.Player->ExplosiveAmmoEnabled && Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->ExplosiveAmmo(true);
		}
		else if (Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->ExplosiveAmmo(false);
		}

		if (ImGui::Checkbox(skCrypt("Fire Ammo"), &g_Config.Player->FireAmmoEnabled)) {}

		if (g_Config.Player->FireAmmoEnabled && Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->FireAmmo(true);
		}
		else if (Core::SDK::Pointers::pLocalPlayer) {
			Core::SDK::Pointers::pLocalPlayer->FireAmmo(false);
		}

		if (ImGui::Checkbox(skCrypt("Infinite Ammo"), &g_Config.Player->InfiniteAmmoEnabled)) {
			if (g_Config.Player->InfiniteAmmoEnabled) {
				Mem.PatchFunc(g_Offsets.m_InfiniteAmmo0, 3);
				Mem.PatchFunc(g_Offsets.m_InfiniteAmmo1, 3);
			}
			else {
				if (!g_Config.Player->NoReloadEnabled) {
					Mem.WriteBytes(g_Offsets.m_InfiniteAmmo0, { 0x41, 0x2B, 0xC9, 0x3B, 0xC8, 0x0F, 0x4D, 0xC8 });
				}
				Mem.WriteBytes(g_Offsets.m_InfiniteAmmo1, { 0x41, 0x2B, 0xD1, 0xE8 });
			}
		}
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Settings"), ImVec2(320, 481));
	{
		ImGui::Spacing();

		static bool waitingforkeyGodMode = false;
		static bool waitingforkeyNoClip = false;
		static bool waitingforkeyFreeCam = false;


		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("GodMode Key"));
		ImGui::SetCursorPosX(20);
		ImGui::Hotkey(&g_Config.Player->GodModeKey, ImVec2(320 - 38, 25), &waitingforkeyGodMode, skCrypt("- GodMode"));

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("NoClip Key"));
		ImGui::SetCursorPosX(20);
		ImGui::Hotkey(&g_Config.Player->NoClipKey, ImVec2(320 - 38, 25), &waitingforkeyNoClip, skCrypt("- NoClip"));

		if (g_Config.Player->NoClipEnabled) {
			ImGui::SliderFloat(skCrypt("NoClip Speed"), &g_Config.Player->NoClipSpeed, 0.1f, 100.f, skCrypt("%1.1fm/s"));
		}

		if (g_Config.Player->FastRun) {
			if (ImGui::SliderFloat(skCrypt("Run Speed"), &g_Config.Player->RunSpeed, 1.f, 10.f, skCrypt("%1.1fm/s"))) {
				Core::SDK::Pointers::pLocalPlayer->SetSpeed(g_Config.Player->FastRun ? g_Config.Player->RunSpeed : 1.f);
			}
		}

		if (g_Config.Player->DamageBoost) {
			if (ImGui::SliderFloat(skCrypt("Damage Boost"), &g_Config.Player->Boost, 1.f, 1000.f, skCrypt("%1.1fm/s"))) {
				Core::SDK::Pointers::pLocalPlayer->GetWeaponManager()->BulletDamage(g_Config.Player->DamageBoost ? g_Config.Player->Boost : 1.f);
			}
		}

		if (g_Config.Player->NoRecoilEnabled) {
			if (ImGui::SliderFloat(skCrypt("Recoil Value"), &g_Config.Player->RecoilValue, 0.0f, 10.f, skCrypt("%1.2fx"))) {
				Core::SDK::Pointers::pLocalPlayer->GetWeaponManager()->SetRecoil(g_Config.Player->RecoilValue);
			}
		}

		if (g_Config.Player->NoSpreadEnabled) {
			if (ImGui::SliderFloat(skCrypt("Spread Value"), &g_Config.Player->SpreadValue, 0.0f, 10.f, skCrypt("%1.2fx"))) {
				Core::SDK::Pointers::pLocalPlayer->GetWeaponManager()->SetSpread(g_Config.Player->SpreadValue);
			}
		}

		g_Config.Player->CurrentHealthValue = Core::SDK::Pointers::pLocalPlayer->GetHealth() - 100.f >
			Core::SDK::Pointers::pLocalPlayer->GetMaxHealth() - 100.f ?
			Core::SDK::Pointers::pLocalPlayer->GetHealth() - 100.f :
			Core::SDK::Pointers::pLocalPlayer->GetHealth() - 99.f;
		g_Config.Player->CurrentArmorValue = Core::SDK::Pointers::pLocalPlayer->GetArmor();

		if (ImGui::SliderFloat(skCrypt("Health Value"), &g_Config.Player->CurrentHealthValue, -1.f,
			Core::SDK::Pointers::pLocalPlayer->GetMaxHealth() - 100.f, skCrypt("%1.f"))) {
			Core::SDK::Pointers::pLocalPlayer->SetHealth(g_Config.Player->CurrentHealthValue + 10000.f);
		}

		if (ImGui::SliderFloat(skCrypt("Armor Value"), &g_Config.Player->CurrentArmorValue, 0.f, 40000.f, skCrypt("%1.f"))) {
			Core::SDK::Pointers::pLocalPlayer->SetArmor(g_Config.Player->CurrentArmorValue);
		}
	}
}

void teleports_tab() {
	static int selectedItem = 0; // Default to first item (Waypoint)

	struct Locations_t {
		std::string Name;
		D3DXVECTOR3 Coords;
	};

	std::vector<Locations_t> Locations = {
		Locations_t(skCrypt("Waypoint").decrypt(), D3DXVECTOR3(0, 0, 0)),
		Locations_t(skCrypt("Square").decrypt(), D3DXVECTOR3(156.184, -1043.17, 29.3236)),
		Locations_t(skCrypt("Pier").decrypt(), D3DXVECTOR3(-1847.72, -1223.36, 13.8745)),
		Locations_t(skCrypt("Paleto Bay").decrypt(), D3DXVECTOR3(-397.605, 6047.57, 32.1797)),
		Locations_t(skCrypt("Central Bank").decrypt(), D3DXVECTOR3(221.781, 217.278, 106.705)),
		Locations_t(skCrypt("Cassino").decrypt(), D3DXVECTOR3(885.322, 16.8489, 80.65)),
		Locations_t(skCrypt("Los Santos Airport").decrypt(), D3DXVECTOR3(-975.532, -2880.89, 16.2665)),
		Locations_t(skCrypt("Sandy Shores").decrypt(), D3DXVECTOR3(1681.48, 3251.91, 40.809)),
	};

	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Locations"), ImVec2(320, 481));
	{
		ImGui::Spacing();

		std::vector<const char*> locationNames;
		for (const auto& location : Locations) {
			locationNames.push_back(location.Name.c_str());
		}

		ImGui::SetCursorPosX(20);
		ImGui::ListBox(skCrypt("Locations"), &selectedItem, locationNames.data(), locationNames.size(), 17);
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Settings"), ImVec2(320, 481));
	{
		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		std::string teleportButtonText = std::string(skCrypt("Teleport to ").decrypt()) + Locations[selectedItem].Name;
		if (ImGui::Button(teleportButtonText.c_str(), ImVec2(320 - 38, 30))) {
			if (selectedItem == 0) {
				Core::Features::Exploits::TpToWaypoint();
			}
			else {
				Core::SDK::Pointers::pLocalPlayer->SetPos(Locations[selectedItem].Coords);
			}
		}
	}
	ImGui::EndChild();
}

void configs_tab() {
	static bool waitingForMenuKey = false;

	ImGui::SetCursorPos(ImVec2(169, 38));
	ImGui::MenuChild(skCrypt("Globals"), ImVec2(320, 481));
	{
		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		ImGui::Text(skCrypt("Menu Key"));
		ImGui::SetCursorPosX(20);
		ImGui::Hotkey(&g_Config.General->MenuKey, ImVec2(320 - 38, 25), &waitingForMenuKey, skCrypt("- Menu"));

		ImGui::Spacing();

		if (ImGui::Checkbox(skCrypt("RGB Particles"), &Gui::cOverlay.enableRGBParticles)) {
			if (Gui::cOverlay.enableRGBParticles) {
				Gui::cOverlay.originalParticlesColor = Gui::cOverlay.particlesColor;
			}
			else {
				Gui::cOverlay.particlesColor = Gui::cOverlay.originalParticlesColor;
			}
		}
		if (!Gui::cOverlay.enableRGBParticles) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 38);
			if (ImGui::ColorButton(skCrypt("Particles Color"), Gui::cOverlay.particlesColor)) {
				ImGui::OpenPopup(skCrypt("ParticlesColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("ParticlesColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&Gui::cOverlay.particlesColor,
					ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar |
					ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}

		if (ImGui::Checkbox(skCrypt("RGB Rocket"), &Gui::cOverlay.enableRGBRocket)) {
			if (Gui::cOverlay.enableRGBRocket) {
				Gui::cOverlay.originalRocketColor = Gui::cOverlay.rocketColor;
			}
			else {
				Gui::cOverlay.rocketColor = Gui::cOverlay.originalRocketColor;
			}
		}
		if (!Gui::cOverlay.enableRGBRocket) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 38);
			if (ImGui::ColorButton(skCrypt("Rocket Color"), Gui::cOverlay.rocketColor)) {
				ImGui::OpenPopup(skCrypt("RocketColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("RocketColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&Gui::cOverlay.rocketColor,
					ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar |
					ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}

		if (ImGui::Checkbox(skCrypt("RGB Infos"), &Gui::cOverlay.enableRGBInfos)) {
			if (Gui::cOverlay.enableRGBInfos) {
				Gui::cOverlay.originalInfosColor = Gui::cOverlay.infosColor;
			}
			else {
				Gui::cOverlay.infosColor = Gui::cOverlay.originalInfosColor;
			}
		}
		if (!Gui::cOverlay.enableRGBInfos) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 38);
			if (ImGui::ColorButton(skCrypt("Infos Color"), Gui::cOverlay.infosColor)) {
				ImGui::OpenPopup(skCrypt("InfosColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("InfosColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&Gui::cOverlay.infosColor,
					ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar |
					ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}

		if (ImGui::Checkbox(skCrypt("RGB Sidebar Icons"), &Gui::cOverlay.enableRGBSidebarIcons)) {
			if (Gui::cOverlay.enableRGBSidebarIcons) {
				Gui::cOverlay.originalSidebarIconsColor = Gui::cOverlay.sidebarIconsColor;
			}
			else {
				Gui::cOverlay.sidebarIconsColor = Gui::cOverlay.originalSidebarIconsColor;
			}
		}
		if (!Gui::cOverlay.enableRGBSidebarIcons) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 38);
			if (ImGui::ColorButton(skCrypt("Sidebar Icons Color"), Gui::cOverlay.sidebarIconsColor)) {
				ImGui::OpenPopup(skCrypt("SidebarIconsColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("SidebarIconsColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&Gui::cOverlay.sidebarIconsColor,
					ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar |
					ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}

		if (ImGui::Checkbox(skCrypt("RGB Sidebar Selected Icon"), &Gui::cOverlay.enableRGBSidebarSelectedIcon)) {
			if (Gui::cOverlay.enableRGBSidebarSelectedIcon) {
				Gui::cOverlay.originalSidebarSelectedIconColor = Gui::cOverlay.sidebarSelectedIconColor;
			}
			else {
				Gui::cOverlay.sidebarSelectedIconColor = Gui::cOverlay.originalSidebarSelectedIconColor;
			}
		}
		if (!Gui::cOverlay.enableRGBSidebarSelectedIcon) {
			ImGui::SameLine();
			ImGui::SetCursorPosX(ImGui::GetWindowWidth() - 38);
			if (ImGui::ColorButton(skCrypt("Sidebar Selected Icon Color"), Gui::cOverlay.sidebarSelectedIconColor)) {
				ImGui::OpenPopup(skCrypt("SidebarSIconColorPicker"));
			}
			if (ImGui::BeginPopup(skCrypt("SidebarSIconColorPicker"))) {
				ImGui::ColorPicker4(skCrypt("##picker"), (float*)&Gui::cOverlay.sidebarSelectedIconColor,
					ImGuiColorEditFlags_NoSidePreview | ImGuiColorEditFlags_AlphaBar |
					ImGuiColorEditFlags_NoSmallPreview | ImGuiColorEditFlags_PickerHueWheel);
				ImGui::EndPopup();
			}
		}

		if (Gui::cOverlay.enableRGBParticles || Gui::cOverlay.enableRGBRocket ||
			Gui::cOverlay.enableRGBInfos || Gui::cOverlay.enableRGBSidebarIcons ||
			Gui::cOverlay.enableRGBSidebarSelectedIcon) {
			ImGui::SliderFloat(skCrypt("RGB Speed"), &Gui::cOverlay.rgbSpeed, 0.01f, 1.0f, "%.2f");
		}

		ImGui::Checkbox(skCrypt("Particles"), &Gui::cOverlay.particles);
		if (Gui::cOverlay.particles) {
			ImGui::SliderInt(skCrypt("Particle Count"), &Gui::cOverlay.particleCount, 1, 300);
			ImGui::SliderFloat(skCrypt("Min Speed"), &Gui::cOverlay.minSpeed, 0.1f, 10.0f);
			ImGui::SliderFloat(skCrypt("Max Speed"), &Gui::cOverlay.maxSpeed, Gui::cOverlay.minSpeed, 50.0f);
			ImGui::SliderFloat(skCrypt("Min Radius"), &Gui::cOverlay.minRadius, 0.1f, 5.0f);
			ImGui::SliderFloat(skCrypt("Max Radius"), &Gui::cOverlay.maxRadius, Gui::cOverlay.minRadius, 10.0f);
		}
	}
	ImGui::EndChild();

	ImGui::SetCursorPos(ImVec2(505, 38));
	ImGui::MenuChild(skCrypt("Configs"), ImVec2(320, 481));
	{
		static int selected_cfg = -1;
		static std::vector<std::string> cfg_list;
		static char config_namee[256] = { 0 };
		static bool needs_refresh = true;
		static bool input_active = false;
		static char last_key_pressed = 0;
		static double last_key_time = 0.0;

		if (needs_refresh || (GetTickCount64() % 1000) == 0)
		{
			cfg_list = g_Config.GetConfigs();
			needs_refresh = false;

			if (selected_cfg >= (int)cfg_list.size())
				selected_cfg = cfg_list.empty() ? -1 : cfg_list.size() - 1;
		}

		ImGui::SetCursorPosX(20);
		if (ImGui::Button(xorstr("Open Config Folder"), ImVec2(320 - 38, 30)))
		{
			g_Config.OpenConfigFolder();
		}
		

		if (ImGui::BeginListBox(xorstr("##ConfigList"), ImVec2(320 - 38, 100)))
		{
			for (size_t i = 0; i < cfg_list.size(); i++)
			{
				bool selected = (selected_cfg == (int)i);
				if (ImGui::Selectable(cfg_list[i].c_str(), selected))
				{
					selected_cfg = (int)i;
				}
				if (selected)
					ImGui::SetItemDefaultFocus();
			}
			ImGui::EndListBox();
		}

		ImGui::SetCursorPosX(20);
		ImGui::Text("Config Name:");

		double current_time = ImGui::GetTime();
		ImGuiInputTextFlags flags = ImGuiInputTextFlags_CharsNoBlank;

		if (input_active && (current_time - last_key_time) < 0.1)
		{
			flags |= ImGuiInputTextFlags_ReadOnly;
		}

		ImGui::SameLine();
		if (ImGui::InputText(xorstr("##ConfigName"), config_namee, sizeof(config_namee), flags))
		{
			if (!input_active || strlen(config_namee) == 0 ||
				config_namee[strlen(config_namee) - 1] != last_key_pressed ||
				(current_time - last_key_time) > 0.1)
			{
				input_active = true;
				if (strlen(config_namee) > 0)
				{
					last_key_pressed = config_namee[strlen(config_namee) - 1];
				}
				last_key_time = current_time;
			}
			else
			{
				config_namee[strlen(config_namee) - 1] = '\0';
			}
		}
		else if (!ImGui::IsItemActive())
		{
			input_active = false;
		}

		ImGui::SetCursorPosX(20);
		if (ImGui::Button(xorstr("Create Config"), ImVec2(320 - 38, 30)) && strlen(config_namee) > 0)
		{
			g_Config.CreateConfig(config_namee);

			cfg_list = g_Config.GetConfigs();
			selected_cfg = (int)cfg_list.size() - 1;

			memset(config_namee, 0, sizeof(config_namee));
			input_active = false;
		}

		if (selected_cfg >= 0 && selected_cfg < (int)cfg_list.size())
		{
			ImGui::SetCursorPosX(20);
			if (ImGui::Button(xorstr("Load"), ImVec2(320 - 38, 30)))
			{
				g_Config.ResetConfig();
				g_Config.LoadConfigFromFile(cfg_list[selected_cfg]);
			}

			ImGui::SetCursorPosX(20);
			if (ImGui::Button(xorstr("Save"), ImVec2(320 - 38, 30)))
			{
				g_Config.SaveCurrentConfigToFile(cfg_list[selected_cfg]);
			}
		}

		ImGui::Spacing();

		ImGui::SetCursorPosX(20);
		if (ImGui::Button(skCrypt("Export Config"), ImVec2(320 - 38, 30))) {
			std::thread cfgExport([] {
				std::string CfgMsg = g_Config.SaveCurrentConfig(skCrypt("...").decrypt());
				NotifyManager::Send(CfgMsg.c_str(), 3000);
				});
			cfgExport.detach();
		}

		ImGui::SetCursorPosX(20);
		if (ImGui::Button(skCrypt("Import Config"), ImVec2(320 - 38, 30))) {
			std::thread cfgImport([] {
				NotifyManager::Send(g_Config.LoadCfg(skCrypt("...").decrypt(), Utils::GetClipboard()).c_str(), 3000);
				});
			cfgImport.detach();
		}

		ImGui::SetCursorPosX(20);
		if (ImGui::Button(skCrypt("Unload"), ImVec2(320 - 38, 30))) {
			exit(0);
		}

		ImGui::Checkbox(skCrypt("VSync"), &g_Config.General->VSync);

		if (ImGui::Checkbox(skCrypt("Stream Proof"), &g_Config.General->StreamProof)) {
			if (g_Config.General->StreamProof) {
				SetWindowDisplayAffinity(g_Variables.g_hCheatWindow, WDA_EXCLUDEFROMCAPTURE);
			}
			else {
				SetWindowDisplayAffinity(g_Variables.g_hCheatWindow, WDA_NONE);
			}
		}
	}
	ImGui::EndChild();
}

static int m_tabs;

void RenderTab()
{
	static int Pad = 4; // Reduzindo a margem
	ImGui::SetCursorPos(ImVec2(45, 12)); // Ajustando posição dentro da sidebar
	ImGui::Image(g_Variables.Logo, ImVec2(70, 70)); // Reduzindo tamanho da logo

	auto draw = ImGui::GetWindowDrawList();
	ImVec2 pos = ImGui::GetWindowPos();

	draw->AddText(poppins, 17, ImVec2(pos.x + 13, pos.y + 81), ImColor(105, 105, 105, int(255 * ImGui::GetStyle().Alpha)), skCrypt("Aim"));
	draw->AddText(poppins, 17, ImVec2(pos.x + 13, pos.y + 210), ImColor(105, 105, 105, int(255 * ImGui::GetStyle().Alpha)), skCrypt("Visuals"));
	draw->AddText(poppins, 17, ImVec2(pos.x + 13, pos.y + 348), ImColor(105, 105, 105, int(255 * ImGui::GetStyle().Alpha)), skCrypt("Miscellaneous"));

	ImGui::SetCursorPos(ImVec2(13, 99));
	if (ImGui::Rendertab("r", skCrypt("Aimbot"), !m_tabs)) m_tabs = 0;

	ImGui::SetCursorPos(ImVec2(13, 136));
	if (ImGui::Rendertab("e", skCrypt("Silent Aim"), m_tabs == 1)) m_tabs = 1;

	ImGui::SetCursorPos(ImVec2(13, 174));
	if (ImGui::Rendertab("a", skCrypt("Trigger Bot"), m_tabs == 2)) m_tabs = 2;

	ImGui::SetCursorPos(ImVec2(13, 228));
	if (ImGui::Rendertab("x", skCrypt("Players"), m_tabs == 3)) m_tabs = 3;

	ImGui::SetCursorPos(ImVec2(13, 266));
	if (ImGui::Rendertab("v", skCrypt("Vehicles"), m_tabs == 4)) m_tabs = 4;

	ImGui::SetCursorPos(ImVec2(13, 304));
	if (ImGui::Rendertab("w", skCrypt("World"), m_tabs == 5)) m_tabs = 5;

	ImGui::SetCursorPos(ImVec2(13, 369));
	if (ImGui::Rendertab("z", skCrypt("Exploits"), m_tabs == 6)) m_tabs = 6;

	ImGui::SetCursorPos(ImVec2(13, 407));
	if (ImGui::Rendertab("s", skCrypt("Teleport"), m_tabs == 7)) m_tabs = 7;

	ImGui::SetCursorPos(ImVec2(13, 445));
	if (ImGui::Rendertab("c", skCrypt("Configs"), m_tabs == 8)) m_tabs = 8;

	switch (m_tabs)
	{
	case 0: legit_tab(); break;
	case 1: silent_tab(); break;
	case 2: trigger_tab(); break;
	case 3: players_tab();  break;
	case 4: vehicles_tab(); break;
	case 5: world_tab(); break;
	case 6: exploits_tab(); break;
	case 7: teleports_tab();  break;
	case 8: configs_tab(); break;
	}
}

void user_info()
{
	auto draw = ImGui::GetWindowDrawList();
	ImVec2 pos = ImGui::GetWindowPos();

	draw->AddRectFilled(ImVec2(pos.x + 9, pos.y + 486), ImVec2(pos.x + 152, pos.y + 523),
		ImColor(41, 41, 41, int(255 * ImGui::GetStyle().Alpha)), 5.f, ImDrawCornerFlags_All);
	draw->AddRect(ImVec2(pos.x + 9, pos.y + 486), ImVec2(pos.x + 152, pos.y + 523),
		ImColor(50, 50, 50, int(255 * ImGui::GetStyle().Alpha)), 5.f, ImDrawCornerFlags_All, 1);

	ImColor infosColorWithAlpha = ImColor(Gui::cOverlay.infosColor.Value.x,
		Gui::cOverlay.infosColor.Value.y,
		Gui::cOverlay.infosColor.Value.z,
		Gui::cOverlay.infosColor.Value.w * ImGui::GetStyle().Alpha);

	std::stringstream discordName;
	discordName << "(" << g_Variables.UserName << ")";
	draw->AddColoredText(ImVec2(pos.x + 24, pos.y + 488),
		ImColor(105, 105, 105, int(255 * ImGui::GetStyle().Alpha)),
		infosColorWithAlpha,
		discordName.str().c_str());

	std::stringstream licenseName;
	licenseName << skCrypt("Left: (").decrypt();

	/*if (g_Api.client.sub_type.expire_date.empty()) {*/
		licenseName << "N/A)";
	/*}
	else {
		try {
			time_t expiry_time = std::stoll(g_Api.client.sub_type.expire_date);
			time_t current_time = std::time(nullptr);
			int days_remaining = static_cast<int>((expiry_time - current_time) / (60 * 60 * 24));

			if (days_remaining > 366) {
				licenseName << "Lifetime)";
			}
			else if (days_remaining >= 0) {
				licenseName << days_remaining << " days)";
			}
			else {
				licenseName << "Expired)";
			}
		}
		catch (const std::exception& e) {
			licenseName << "Error: " << e.what() << ")";
		}
	}*/

	draw->AddColoredText(ImVec2(pos.x + 24, pos.y + 503),
		ImColor(105, 105, 105, int(255 * ImGui::GetStyle().Alpha)),
		infosColorWithAlpha,
		licenseName.str().c_str());
}

void Decoration()
{
	auto draw = ImGui::GetWindowDrawList();
	ImVec2 pos = ImGui::GetWindowPos();

	draw->AddRectFilled(ImVec2(pos.x, pos.y), ImVec2(pos.x + 161, pos.y + 535), ImColor(22, 22, 22, int(255 * ImGui::GetStyle().Alpha)), 10.f, ImDrawCornerFlags_Left); // left bg
	draw->AddRect(ImVec2(pos.x, pos.y), ImVec2(pos.x + 161, pos.y + 535), ImColor(50, 50, 50, int(255 * ImGui::GetStyle().Alpha)), 10.f, ImDrawCornerFlags_Left, 1);  // left bg

	draw->AddRectFilled(ImVec2(pos.x + 160, pos.y), ImVec2(pos.x + 838, pos.y + 535), ImColor(18, 18, 18, int(255 * ImGui::GetStyle().Alpha)), 10.f, ImDrawCornerFlags_Right); // right bg
	draw->AddRect(ImVec2(pos.x + 160, pos.y), ImVec2(pos.x + 838, pos.y + 535), ImColor(50, 50, 50, int(255 * ImGui::GetStyle().Alpha)), 10.f, ImDrawCornerFlags_Right, 1);  // right bg

	ImColor rocketColorWithAlpha = ImColor(Gui::cOverlay.rocketColor.Value.x, Gui::cOverlay.rocketColor.Value.y, Gui::cOverlay.rocketColor.Value.z, Gui::cOverlay.rocketColor.Value.w * ImGui::GetStyle().Alpha);
	ImColor infosColorWithAlpha = ImColor(Gui::cOverlay.infosColor.Value.x, Gui::cOverlay.infosColor.Value.y, Gui::cOverlay.infosColor.Value.z, Gui::cOverlay.infosColor.Value.w * ImGui::GetStyle().Alpha);
	//draw->AddText(baloo, 38, ImVec2(pos.x + 24, pos.y + 24), rocketColorWithAlpha, "");
	ImGui::SetCursorPos(ImVec2(34, 0));
	//ImGui::Image(g_Variables.Logo, ImVec2(80, 80));

	std::stringstream versionInfo;
	versionInfo << skCrypt("Ver: (").decrypt() << std::fixed << std::setprecision(1) << "1.0" << ")";
	draw->AddColoredText(ImVec2(pos.x + 780, pos.y + 515), ImColor(105, 105, 105, int(255 * ImGui::GetStyle().Alpha)), infosColorWithAlpha, versionInfo.str().c_str());
}

void Render()
{
	ImGui::SetNextWindowSize(ImVec2(838 * 1.0f, 535 * 1.0f));
	ImGui::PushStyleVar(ImGuiStyleVar_Alpha, Gui::cOverlay.backgroundAlpha);
	ImGui::Begin(skCrypt(""), nullptr, ImGuiWindowFlags_NoDecoration | ImGuiWindowFlags_NoSavedSettings | ImGuiWindowFlags_NoBackground);
	{
		Decoration();
		RenderTab();

		Particles();
	}
	ImGui::End();

	ImGui::PushStyleVar(ImGuiStyleVar_WindowRounding, 5.f);
	ImGui::PushStyleColor(ImGuiCol_WindowBg, ImVec4(43.f / 255.f, 43.f / 255.f, 43.f / 255.f, 100.f / 255.f));
	ImGui::PopStyleVar(1);
	ImGui::PopStyleColor(1);

	float time = ImGui::GetTime();
	ImColor rgbColor = ImColor::HSV(fmod(time * Gui::cOverlay.rgbSpeed, 1.0f), 0.8f, 0.8f);

	if (Gui::cOverlay.enableRGBParticles) {
		Gui::cOverlay.particlesColor = rgbColor;
	}
	if (Gui::cOverlay.enableRGBRocket) {
		Gui::cOverlay.rocketColor = rgbColor;
	}
	if (Gui::cOverlay.enableRGBInfos) {
		Gui::cOverlay.infosColor = rgbColor;
	}
	if (Gui::cOverlay.enableRGBSidebarIcons) {
		Gui::cOverlay.sidebarIconsColor = rgbColor;
	}
	if (Gui::cOverlay.enableRGBSidebarSelectedIcon) {
		Gui::cOverlay.sidebarSelectedIconColor = rgbColor;
	}

}

void Gui::Rendering()
{
	if (!g_MenuInfo.IsLogged)
	{
		g_MenuInfo.MenuSize = { 500, 350 };
	}
	ImGui::SetNextWindowSize(g_MenuInfo.MenuSize);

	if (!PauseLoop) { ImGui::SetNextWindowPos(g_Variables.g_vGameWindowSize / 2 - g_MenuInfo.MenuSize / 2); PauseLoop++; }

	ImGui::PushFont(g_Variables.m_FontNormal);

	if (g_MenuInfo.IsOpen)
		Render();

	HWND ActiveWindow = GetForegroundWindow();

	{
		std::lock_guard<std::mutex> Lock(DrawMtx);

		NotifyManager::Render();

		if (ActiveWindow == g_Variables.g_hGameWindow)
		{
			if (GetAsyncKeyState(g_Config.Player->GodModeKey) & 1)
			{
				g_Config.Player->EnableGodMode = !g_Config.Player->EnableGodMode;

				Core::SDK::Pointers::pLocalPlayer->SetGodMode(g_Config.Player->EnableGodMode);

				std::thread([&]()
					{
						NotifyManager::Send(xorstr("GodMode has been ") + (std::string)(g_Config.Player->EnableGodMode ? xorstr("enabled!") : xorstr("disabled!")), 2000);
					}
				).detach();

			}

			if (GetAsyncKeyState(g_Config.ESP->KeyBind) & 1)
			{
				g_Config.ESP->Enabled = !g_Config.ESP->Enabled;

				std::thread([&]()
					{
						NotifyManager::Send(xorstr("ESP has been ") + (std::string)(g_Config.ESP->Enabled ? xorstr("enabled!") : xorstr("disabled!")), 2000);
					}
				).detach();
			}

			if (GetAsyncKeyState(g_Config.Player->NoClipKey) & 1)
			{
				g_Config.Player->NoClipEnabled = !g_Config.Player->NoClipEnabled;

				Core::SDK::Pointers::pLocalPlayer->FreezePed(g_Config.Player->NoClipEnabled);

				std::thread([&]()
					{
						NotifyManager::Send(xorstr("NoClip has been ") + (std::string)(g_Config.Player->NoClipEnabled ? xorstr("enabled!") : xorstr("disabled!")), 2000);
					}
				).detach();
			}

			if (GetAsyncKeyState(g_Config.Player->FreeCamKey) & 1)
			{
				g_Config.Player->FreeCam = !g_Config.Player->FreeCam;
			}

			if (g_Config.Player->NoClipEnabled)
				Features::Exploits::NoClip();
		}

		if (ActiveWindow == g_Variables.g_hGameWindow || ActiveWindow == g_Variables.g_hCheatWindow)
		{

			struct FovFuncs_t {
				bool* Enabled;
				int* FovSize;
				ImVec4 FovColor;
			};

			std::vector<FovFuncs_t> FovDrawList = {
				FovFuncs_t(&g_Config.Aimbot->ShowFov, &g_Config.Aimbot->FOV, g_Config.Aimbot->FovColor),
				FovFuncs_t(&g_Config.SilentAim->ShowFov, &g_Config.SilentAim->FOV, g_Config.SilentAim->FovColor),
				FovFuncs_t(&g_Config.TriggerBot->ShowFov, &g_Config.TriggerBot->FOV, g_Config.TriggerBot->FovColor),
			};

			static std::vector<float> Alphas(FovDrawList.size(), 0.0f);
			static std::vector<float> Sizes(FovDrawList.size(), 0.0f);

			for (int i = 0; i < FovDrawList.size(); ++i)
			{
				auto& Fov = FovDrawList[i];

				Alphas[i] = ImClamp(ImLerp(Alphas[i], *Fov.Enabled ? 1.f : 0.f, ImGui::GetIO().DeltaTime * 10.f), 0.f, 1.f);
				Sizes[i] = ImLerp(Sizes[i], (float)*Fov.FovSize, ImGui::GetIO().DeltaTime * 12.f);

				ImGui::PushStyleVar(ImGuiStyleVar_Alpha, Alphas[i]);

				ImGui::GetBackgroundDrawList()->AddCircle(ImVec2(g_Variables.g_vGameWindowCenter.x, g_Variables.g_vGameWindowCenter.y), Sizes[i], ImGui::GetColorU32(Fov.FovColor), 999);

				ImGui::PopStyleVar();
			}

			if (g_Config.General->WaterMark)
				Custom::WaterMark::Render();

			ImGui::PushFont(g_Variables.m_DrawFont);

			Features::g_Esp.Draw();
			Features::g_Esp.DrawVehicle();

			ImGui::PopFont();
		}
		else {
			if (ImGui::GetStyle().Alpha >= 0.9f)
			{
				g_MenuInfo.IsOpen = false;
				SetWindowLong(g_Variables.g_hCheatWindow, GWL_EXSTYLE, WS_EX_TOPMOST | WS_EX_LAYERED | WS_EX_TOOLWINDOW | WS_EX_TRANSPARENT);
			}
		}

	}

	ImGui::PopFont();
}
