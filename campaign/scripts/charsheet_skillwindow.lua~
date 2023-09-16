-- 
-- Please see the license.html file included with this distribution for 
-- attribution and copyright information.
--
bIsRefresh = false;

function getCategory()
	local skilltype = type.getValue();
	if group.getValue()~="" then
		return group.getValue();
	end
	if skilltype==1 then
		return "Maneuver, Moving";
	elseif skilltype==2 then
		return "Maneuver, Static";
	elseif skilltype==3 then
		return "Offense Bonus";
	elseif skilltype==4 then
		return "Special";
	else
		return "Other";
	end
end

local initialised = false;

function refresh()
	local clc = calc.getState();
	local rnk = rank.getValue();
	--[[ Don't proceed if the window isn't fully initialised ]]
	if not initialised then
		return;
	end
	if not bIsRefresh then
		bIsRefresh = true;
		--[[ Get the average bonus from all applicable stats ]]
		local charNode = getDatabaseNode().getParent().getParent();
		statbonus.setValue(Rules_PC.CombinedStatBonus(charNode, stats.getValue(), fullname.getValue()));
		--[[ Set the rank bonus, if needed ]]
		if clc==1 or clc==2 then
			local rkbn = getRankBonus(rnk,clc);
			rankbonus.setReadOnlyRMC(true);
			rankbonus.setFrame(nil);
			total.setReadOnlyRMC(true);
			total.setFrame(nil);
			rankbonus.setValue(rkbn);
			
			total.setValue(rkbn + statbonus.getValue() + level.getValue() +
						   item.getValue() + special.getValue() + misc.getValue());
		elseif clc==3 then
			-- rkbn is your "rolled" hits
			local rkbn = rankbonus.getValue();
			-- bht is your temp Con / 10
			local bht = Rules_PC.BHT(charNode);
			bht = bht + rkbn;
			rankbonus.setReadOnlyRMC(false);
			rankbonus.setFrame("textline",0,5,0,0);
			total.setReadOnlyRMC(true);
			total.setFrame(nil);
			-- BHT + (BHT x (Con Bonus/100)) round down
			total.setValue(math.floor(bht + (bht * (statbonus.getValue()/100)) + level.getValue() +
						   item.getValue() + special.getValue() + misc.getValue()));
		elseif clc==5 then
			local rkbn = rnk;
			rankbonus.setReadOnlyRMC(true);
			rankbonus.setFrame(nil);
			rankbonus.setValue(rnk);
			statbonus.setReadOnlyRMC(true);
			statbonus.setFrame(nil);
			statbonus.setValue(Rules_PC.RealmPPMultiplier(windowlist.window.getDatabaseNode()));
			total.setReadOnlyRMC(true);
			total.setFrame(nil);
			total.setValue(math.floor(rkbn * statbonus.getValue()) + level.getValue() +
						   item.getValue() + special.getValue() + misc.getValue());
		else
			rankbonus.setReadOnlyRMC(false);
			rankbonus.setFrame("textline",0,5,0,0);
			total.setReadOnlyRMC(false);
			total.setFrame("textline",0,5,0,0);
		end
		shortname.update();
		checkCost();
		setMenus();
		bIsRefresh = false;
	end
end

function getRankBonus(rank,calc)
	return Rules_Skills.RankBonus(rank, calc);
end

function checkCost()
	local skill = name.getValue();
	-- is there a name yet?
	if skill=="" then
		return;
	end
	-- is there a development cost recorded?
	if cost.getValue()=="" then
		-- Set the development cost
		cost.setValue(windowlist.getCost(skill));
	end
end

function onInit()
	if super and super.onInit then
		super.onInit();
	end
	name.getDatabaseNode().onUpdate = refresh;
	stats.getDatabaseNode().onUpdate = refresh;
	rank.getDatabaseNode().onUpdate = refresh;
	level.getDatabaseNode().onUpdate = refresh;
	item.getDatabaseNode().onUpdate = refresh;
	special.getDatabaseNode().onUpdate = refresh;
	misc.getDatabaseNode().onUpdate = refresh;
	windowlist.window.getDatabaseNode().getChild(".abilities").onChildUpdate = refresh;

	updateNotesField();
	-- done
	initialised = true;
	refresh();
end

function updateNotesField()
	local node = getDatabaseNode();
	if node.getChild("notes") and node.getChild("notes").getType() == "formattedtext" then
		local sNotes = DB.getValue(node, "notes", "");
		sNotes = string.gsub(sNotes, "<p>", "");
		sNotes = string.gsub(sNotes, "</p>", "");
		DB.deleteNode(node.getChild("notes"));
		node.createChild("notes","string").setValue(sNotes);
	end
end
          
function setMenus()
	resetMenuItems();
	-- delete menu item
	registerMenuItem("Delete Skill","delete",6);
end

function onMenuSelection(item)
	if item then
		if item==6 then
			getDatabaseNode().delete();
		end
	end
end

function parseShortName()
	--[[ extract the stats and the skill name from the shortname control]]
	local s = shortname.getValue();
	local i = string.find(s,"(",1,true);
	local nm = "";
	local sts = "";
	if i and i > 0 then
		local j = string.find(s,")",i+1,true);
		if j and j > 0 then
			sts = string.sub(s,i+1,j-1);
		else
			sts = string.sub(s,i+1);
		end
		nm = string.gsub(string.sub(s,1,i-1),"%s+$","");
	else
		nm = s;
		sts = "";
	end
	--[[ update the name and stat fields, if needed ]]
	if name and name.getValue()~=nm then
		name.setValue(nm);
    end
	if stats and stats.getValue()~=sts then
		stats.setValue(sts);
	end
	--[[ done ]]
	return;
end
