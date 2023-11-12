--____GLOBALS FOR ELE ROTATION_______
local elementalFocusBuffIcon = "Interface\Icons\Spell_Shadow_ManaBurn"
local elementalMasteryBuffIcon = "Interface\Icons\Spell_Nature_WispHeal"
local manaConsumesThresholds = { 1600, 2100 }
local manaConsumes = { "Nordanaar Herbal Tea", "Major Mana Potion" }
local equippedSlotsToUse = { 13, 14 }
local chainLightningManaCost = 544

--_____MAIN______
function EleShamanRotation(fullDmg, useCons, useChainLightning, useEquippedSlots)
  if (UsedManaConsumes(useCons)) then
    return 1
  end
  if (not ElementalMasteryOnCooldown()) then
    cast("Elemental Mastery")
    return 1
  end
  if (UsedTrinkets(useEquippedSlots)) then
    return 1
  end
  if (ShouldUseChainLightning(fullDmg, useChainLightning)) then
    cast("Chain Lightning")
  else
    cast("Lightning Bolt")
  end
  return 1
end

--________ELE ROTATION FUNCTIONS_________

function ShouldUseChainLightning(fullDmg, useChainLightning)
  if (not useChainLightning) then
    return false
  end
  local spellId = GetSpellId("Chain Lightning", "Rank 4")
  if (SpellOnCooldown(spellId)) then
    return false
  end
  local elementalFocus = GetBuffIndex(elementalFocusBuffIcon)
  local manaForChainLighting = HasManaForChainLightning(elementalFocus)
  if (fullDmg and manaForChainLighting) then
    return true
  end
  local elementalMastery = GetBuffIndex(elementalMasteryBuffIcon)
  if ((elementalFocus > -1 and manaForChainLighting) or elementalMastery > -1) then
    return true
  end
  return false
end

function HasManaForChainLightning(elementalFocus)
  local cost = chainLightningManaCost
  if (elementalFocus > -1) then
    cost = cost * 0.6
  end
  return UnitMana("player") > cost
end

function ElementalMasteryOnCooldown()
  local spellId = GetSpellId("Elemental Mastery", "")
  if (SpellOnCooldown(spellId)) then
    return true
  end
  return false
end

function UsedManaConsumes(useCons)
  if (not useCons) then
    return false
  end
  for i=1, CountElements(manaConsumes), 1 do
    missingMana = GetMissingMana()
    if (missingMana > manaConsumesThresholds[i] and not BagItemOnCooldown(manaConsumes[i])) then
      local used = UseBagItem(manaConsumes[i])
      if (used) then
        return true
      end
    end
  end
  return false
end

function UsedTrinkets(useEquippedSlots)
  if (not useEquippedSlots) then
    return false
  end
  for i=1, CountElements(equippedSlotsToUse), 1 do
    if (not EquippedItemOnCooldown(equippedSlotsToUse[i])) then
      UseEquippedItem(equippedSlotsToUse[i])
      return true
    end
  end
  return false
end

--________GLOBAL FUNCTIONS_________

function Print(text)
  DEFAULT_CHAT_FRAME:AddMessage(text);
end

function PrintAllActiveBuffs()
  local i =1
  while (UnitBuff("player",i)) do
    Print(UnitBuff("player", i))
    i = i+1
  end
end

function GetBuffIndex(buffIcon)
  local i = 1
  while (UnitBuff("player",i)) do
    if (UnitBuff("player",i) == buffIcon) then
      return i -1
    end
    i = i + 1
  end 
  return -1
end

function GetMissingMana()
  return UnitManaMax("player") - UnitMana("player")
end

function GetSpellId(spellName, spellRank)
  local name, rank, i; 
  for i=1, GetSpellBookLength(), 1 do 
    name, rank = GetSpellName(i, "spell")
    if (name and name == spellName and (not spellRank or spellRank == rank)) then 
      return i
    end
  end
  Print(string.format("GetSpellId(spellName, spellRank) could not find id for: '%s %s'", spellName, spellRank))
  return -1
end

function SpellOnCooldown(spellId)
  local success, start, duration, enabled = pcall(GetSpellCooldown, spellId, "spell")
  if (not success) then
    Print(string.format("SpellOnCooldown(spellId) could not find spell with id: '%s'", tostring(spellId)))
    return true
  end
  if ((start > 0 and duration > 0) or enabled == 0) then
    return true
  end
  return false
end

function BagItemOnCooldown(itemName)
  local bag, slot = FindItem(itemName)
  if (bag == nil) then
    Print(string.format("BagItemOnCooldown(itemName) could not find item: '%s'", itemName))
    return true
  end
  local start, duration, enabled = GetContainerItemCooldown(bag, slot)
  if ((start > 0 and duration > 0) or enabled == 0) then
    return true
  end
  return false
end

function UseBagItem(itemName)
  local bag, slot = FindItem(itemName)
  if (bag == nil) then
    Print(string.format("UseBagItem(itemName) could not find item: '%s'", itemName))
    return false
  end
  UseContainerItem(bag, slot)
  return true
end

function EquippedItemOnCooldown(itemSlotIndex)
  local success, start, duration, enabled = pcall(GetInventoryItemCooldown, "player", itemSlotIndex)
  if (not success) then
    Print(string.format("EquippedItemOnCooldown(itemSlotIndex) could not find itemSlot with id: '%s'", tostring(spellId)))
    return true
  end
  if ((start > 0 and duration > 0) or enabled == 0) then
    return true
  end
  return false
end

function UseEquippedItem(itemSlotIndex)
  UseInventoryItem(itemSlotIndex);
  return true
end

function GetSpellBookLength()
  local i = 1
  while true do
    local spellName, spellRank = GetSpellName(i, "spell")
    if (not spellName) then
      break 
    end
    i = i + 1
  end
  return i
end

function PrintAllSpellRanks(amount)
  local i = 1
  while true do
    local spellName, spellRank = GetSpellName(i, "spell")
    Print(string.format("'%s %s'", spellName, spellRank))
    if(not spellName) then
      break 
    end
    i = i + 1
    if (i > amount) then
      break
    end
  end
  return i
end

function CountElements(table)
  local count = 0
  for _ in pairs(table) do
    count = count + 1
  end
  return count
end
