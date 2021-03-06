# Dosage instructions

The dosage instructions tell the patient when and how to take the medication.
[ArtDecor template](https://art-decor.org/art-decor/decor-templates--cdachemed-?id=2.16.756.5.30.1.1.10.4.35).

## Dose regime

<span class="must-support">Must support</span>.
The dose regime is described by an additional templateId in the `#!xml <hl7:substanceAdministration>` or the `#!xml <hl7:supply>`.

!!! bug inline end "Not fully supported"

    Only the normal and split regimes are supported as of now.

| Regime | templateId | Description |
| ----------------- | ------------------------------- | ----------- |
| Structured normal | 1.3.6.1.4.1.19376.1.5.3.1.4.7.1 | Daily intake(s) with a single quantity |
| Narrative normal  | 1.3.6.1.4.1.19376.1.5.3.1.4.7.1 | Narrative instructions |
| Tapered doses     | 1.3.6.1.4.1.19376.1.5.3.1.4.8   | N/A |
| Split doses       | 1.3.6.1.4.1.19376.1.5.3.1.4.9   | Daily intakes with differents quantities |
| Conditional doses | 1.3.6.1.4.1.19376.1.5.3.1.4.10  | N/A |
| Combination medication components | 1.3.6.1.4.1.19376.1.5.3.1.4.11 | N/A |

In normal dose regime, the dosage instructions can be given in structured form (with `#!xml <hl7:effectiveTime>`, `#!xml <hl7:routeCode>`, `#!xml <hl7:approachSiteCode>`, `#!xml <hl7:doseQuantity>` and `#!xml <hl7:rateQuantity>`) or in narrative form (with the appropriate `#!xml <hl7:entryRelationship>`), but not both at the same time.

## Start and stop

<span class="must-support">Must support</span>.
The intake start and stop date (with or without time) is given by the first `#!xml <hl7:effectiveTime>`, with type `IVL<TS.CH.TZ>`. Boundaries are inclusive. The dates SHOULD be precise to the day (`YYYYMMDD`); there SHOULD be a good reason to be more precise than that.
<!-- TODO optional in split, but cannot be redefined in the related components -->

## Medication frequency

The moment(s) of the day at which the patient is to take the medication.

!!! warning "CARA: additional requirement"

    The eMedication services restricts the event to the [TimingEvent (Ambu) value set](https://art-decor.org/art-decor/decor-valuesets--cdachemed-?id=2.16.756.5.30.1.127.77.12.11.2).
    
    | Event   | Description |
    | ------- | ----------- |
    | `MORN`  | Morning     |
    | `NOON`  | Noon        |
    | `EVE`   | Evening     |
    | `NIGHT` | Night       |

<!-- TODO Definition -->

=== "Single event time interval"
    
    This is only used with the structured normal regime or split regime to define a single daily intake of the dose given by `#!xml <hl7:doseQuantity>`; it's forbidden in narrative dose regime. This is the `#!xml <hl7:effectiveTime>` with type `EIVL<TS>`.

    ```xml title="Example usage of single event"
    <hl7:effectiveTime type="EIVL_TS" operator="A">
        <hl7:event code="EVE" /> <!--(1)-->
    </hl7:effectiveTime>
    ```

    1.  The patient must take the dose given by the `#!xml <hl7:doseQuantity>` element once by day in the evening.
    
=== "Multiple events time interval"

    This is only used with the structured normal regime to define multiple daily intakes of the same dose (given by `#!xml <hl7:doseQuantity>`); it's forbidden in narrative and split dose regimes. This is the `#!xml <hl7:effectiveTime>` with type `SXPR<TS>`.

    ```xml title="Example usage of multiple events"
    <hl7:effectiveTime type="SXPR_TS" operator="A">
        <hl7:comp type="EIVL_TS">
            <hl7:event code="MORN" />
        </hl7:comp>
        <hl7:comp type="EIVL_TS">
            <hl7:event code="NON" />
        </hl7:comp>
        <hl7:comp type="EIVL_TS">
            <hl7:event code="NIGHT" /> <!--(1)-->
        </hl7:comp>
    </hl7:effectiveTime>
    ```

    1.  The patient must take the dose given by the `#!xml <hl7:doseQuantity>` element each day in the morning, at noon and at night.

## Repeat number

The repeat number is mandatory and can be used to limit the number of allowed refills of the prescription item. It does not count the initial dispense, only the following ones (refills).

```xml title="Usage of the repeat number"
<hl7:repeatNumber value="0" /><!-- No refill is allowed, only the initial dispense -->
<hl7:repeatNumber value="2" /><!-- Two refills are allowed (total: three dispenses) -->
<hl7:repeatNumber nullFlavor="NI" /><!-- The refills are unlimited -->
```

The refill quantity is the same as the one provided in the item entry.

## Route of administration

<span class="should-support">Should support</span>.
!!! warning "CARA: additional requirement"

    The eMedication service restricts the unit to the ["RouteOfAdministration (Ambu)" value set](https://art-decor.org/art-decor/decor-valuesets--cdachemed-?id=2.16.756.5.30.1.1.11.3).

## Approach site code

!!! bug "Not implemented"

    This is not yet described in CDA-CH-EMED. A value set of common sites is to be created and adopted by eHealthSuisse.

## Dose quantity

<span class="must-support">Must support</span>.
The quantity of medication to take at each intake. 

The value SHALL be a positive, non-zero, non-infinite decimal value or a double.

1.  A decimal value can be "1.23", "21", "1.0". See the [XML Schema decimal specification](https://www.w3.org/TR/xmlschema-2/#decimal).
2.  A double value can be "1", "3E2". See the [XML Schema double specification](https://www.w3.org/TR/xmlschema-2/#double).

!!! warning "CARA: additional requirements"

    The eMedication service restricts the unit to the ["RegularUnitCode (Ambu)" value set](https://art-decor.org/art-decor/decor-valuesets--cdachemed-?id=2.16.756.5.30.1.127.77.12.11.3).<br>
    Furthemore, the dose SHALL be a simple quantity, not a range.

!!! error "Please avoid"

    Please avoid using values that can confuse the reader, are weird or do not make sense, as "0.333", "0.42", "1.618", "INF", "-1E4", "0", "12678967.543233".

```xml title="Examples"
<hl7:doseQuantity value="1" unit="732936001" /><!-- One tablet -->
<hl7:doseQuantity value="10" unit="732994000" /><!-- Ten drops -->
<hl7:doseQuantity value="0.5" unit="[tsp_m]" /><!-- Half a teaspoon -->
```

## Rate quantity

!!! bug "Not implemented"

    This is not yet described in CDA-CH-EMED. As it mainly concerns perfusions and are uncommon for outpatients, it's not a priority to support it. You SHOULD NOT include it.

## Narrative dosage instructions

<span class="must-support">Must support</span>.
The narrative dosage instructions can only appear in the normal dose regime.

```xml title="Usage of the narrative dosage instructions"
<hl7:entryRelationship typeCode="COMP">
    <hl7:substanceAdministration moodCode="INT" classCode="SBADM">
        <hl7:templateId root="2.16.756.5.30.1.1.10.4.52" />
        <hl7:text>
            Take one pill in the morning and two pills in the evening with some water.
            <hl7:reference value="#dosageinstructions" />
        </hl7:text>
        <hl7:consumable>
            <hl7:manufacturedProduct>
                <hl7:manufacturedMaterial nullFlavor="NA" />
            </hl7:manufacturedProduct>
        </hl7:consumable>
    </hl7:substanceAdministration>
</hl7:entryRelationship>
```

## Related components

<span class="must-support">Must support</span>.
They are used in the split dose regime and are forbidden otherwise. It consists of `#!xml <hl7:entryRelationship>`s that contain a `#!xml <hl7:sequenceNumber>` and a `#!xml <hl7:substanceAdministration>`. The value of the sequence number SHALL be an ordinal number, starting at 1 for the first component, and increasing by 1 for each subsequent component. Components SHALL be sent in the sequence number order.

```xml title="Structure of related components"
<entryRelationship typeCode="COMP">
    <sequenceNumber value="1" />
    <substanceAdministration moodCode="INT" classCode="SBADM">
        <!-- effectiveTime[type="EIVL_TS"] and/or doseQuantity -->
        <consumable>
            <manufacturedProduct>
                <manufacturedMaterial nullFlavor="NA" />
            </manufacturedProduct>
        </consumable>
    </substanceAdministration>
</entryRelationship>
<entryRelationship typeCode="COMP">
    <sequenceNumber value="2" />
    <substanceAdministration moodCode="INT" classCode="SBADM">
        <!-- effectiveTime[type="EIVL_TS"] and/or doseQuantity -->
        <consumable>
            <manufacturedProduct>
                <manufacturedMaterial nullFlavor="NA" />
            </manufacturedProduct>
        </consumable>
    </substanceAdministration>
</entryRelationship>
```

The `#!xml <hl7:substanceAdministration>`s contain a null-flavored `#!xml <hl7:substanceAdministration>` and may contain an `#!xml <hl7:effectiveTime>` and/or `#!xml <hl7:doseQuantity>` (as described in "Single event time interval" and "Dose quantity").
If absent, the value of the equivalent element in the parent `#!xml <hl7:substanceAdministration>` applies.
Document creator SHOULD omit the parent elements and always provide both elements in the components.

## Examples

=== "Normal with structured instructions"

    ```xml
    
    ```

=== "Normal with narrative instructions"

    ```xml
    
    ```

=== "Split dose"
    
    ```xml
    <substanceAdministration classCode="SBADM" moodCode="INT">
        <templateId root="2.16.756.5.30.1.1.10.4.34" />
        <templateId root="1.3.6.1.4.1.19376.1.9.1.3.7" />
        <templateId root="2.16.840.1.113883.10.20.1.24" />
        <templateId root="1.3.6.1.4.1.19376.1.5.3.1.4.7" />
        <templateId root="1.3.6.1.4.1.19376.1.9.1.3.6" />
        <templateId root="1.3.6.1.4.1.19376.1.5.3.1.4.9" /> <!--(1)-->
        <id root="00000000-0000-0000-0000-000000000002" />
        <text><reference value="#pdf_ref" /></text>
        <statusCode code="completed" />
        <effectiveTime xsi:type="IVL_TS">
            <low value="202201101043+0100" />
            <high nullFlavor="UNK" />
        </effectiveTime>
        <repeatNumber nullFlavor="NI"/>
        <routeCode code="20049000" codeSystem="0.4.0.127.0.16.1.1.2.1" displayName="Nasal use" />
        <consumable>
            <manufacturedProduct>
                <templateId root="1.3.6.1.4.1.19376.1.5.3.1.4.7.2" />
                <templateId root="2.16.840.1.113883.10.20.1.53" />
                <manufacturedMaterial classCode="MMAT" determinerCode="KIND">
                    <templateId root="2.16.756.5.30.1.1.10.4.33" />
                    <templateId root="1.3.6.1.4.1.19376.1.9.1.3.1" />
                    <code code="7680541890365" codeSystem="2.51.1.1" codeSystemName="GTIN" displayName="NASONEX spray nasal doseur 50 mcg" />
                    <name>NASONEX spray nasal doseur 50 mcg</name>
                    <pharm:formCode code="10808000" codeSystem="0.4.0.127.0.16.1.1.2.1" displayName="Nasal spray, solution" />
                </manufacturedMaterial>
            </manufacturedProduct>
        </consumable>
        <entryRelationship typeCode="COMP">
            <sequenceNumber value="1" /><!--(2)-->
            <substanceAdministration moodCode="INT" classCode="SBADM">
                <effectiveTime xsi:type="EIVL_TS">
                    <event code="MORN" />
                </effectiveTime>
                <doseQuantity unit="{Dose}" value="2"/>
                <consumable>
                    <manufacturedProduct>
                        <manufacturedMaterial nullFlavor="NA" />
                    </manufacturedProduct>
                </consumable>
            </substanceAdministration>
        </entryRelationship>
        <entryRelationship typeCode="COMP">
            <sequenceNumber value="2" /> <!--(3)-->
            <substanceAdministration moodCode="INT" classCode="SBADM">
                <effectiveTime xsi:type="EIVL_TS">
                    <event code="EVE" />
                </effectiveTime>
                <doseQuantity unit="{Dose}" value="1"/>
                <consumable>
                    <manufacturedProduct>
                        <manufacturedMaterial nullFlavor="NA" />
                    </manufacturedProduct>
                </consumable>
            </substanceAdministration>
        </entryRelationship>
    </substanceAdministration>
    ```
    
    1.  The template ID for split dose regime.
    2.  Two doses in the morning.
    3.  One dose in the evening.
