# DDT AI Playbook

These guidelines outline the recommended design workflow for projects that will be developed using AI-assisted workflows. Following these standards will help streamline development, reduce revisions, and improve AI interpretation of design files.

---

# Adjusted Design Workflow

## 1. Design as Usual

Proceed with the standard design workflow while keeping the AI optimization guidelines below in mind.

When providing estimates, include additional time required for:

- AI optimization
- Asset preparation
- Layer organization and cleanup
- AI-ready grouping and structure

---

## 2. Finalize Designs First

Ensure the main design is finalized before creating the AI-optimized version.

This avoids maintaining and revising two separate versions simultaneously, which can lead to inconsistencies and unnecessary work.

---

## 3. Create a Proper Style Guide

Establish a complete style guide before preparing the AI-optimized version.

This should include:

- Text styles
- Color styles
- Button and link variants (hover, active, disabled states)
- Components and variants
- Spacing rules
- Common UI elements

Use consistent styles throughout the entire project.

### Reference

- Ride Nagano Style Guide Reference

---

## 4. Create the AI-Optimized Version

Duplicate the finalized prototype and prepare a dedicated AI-optimized version.

This version should contain:

- Optimized assets
- Clean grouping structure per section
- Properly named sections and layers
- Simplified layouts where necessary

### Reference

- Ride Nagano AI-Optimized Reference

---

## 5. Development Handover

Coordinate closely with the development team throughout the handover process.

Within the AI-optimized file, you may also include notes for:

- Animations
- Transitions
- Interactions
- Client requests or designer notes
- Special implementation considerations

---

# Guidelines for the AI-Optimized Version

## Duplicate Version Setup

- Label the duplicated page as **“(AI-Optimized)”**
- Remove unnecessary duplicate frames/pages used only for transitions or presentation purposes
- Keep the file structure clean and focused on implementation

---

## Style Guide Requirements

- Ensure all text styles and color styles are consistently applied
- Components and reusable elements should follow the established design system
- Avoid excessive one-off styling unless necessary

---

## Image Optimization

Optimize all images using:
- https://tinypng.com/
- Or similar optimization tools

Additional guidelines:

- Crop images to their actual displayed frame size whenever possible
- You may use the **“Copy as PNG”** feature to instantly copy the cropped image
- AI tools often detect the original uncropped image dimensions inside frames

---

## Icons & Graphic Elements

- Combine SVG vectors where possible
- Especially vectors sharing the same color or structure
- Flatten or group decorative graphic elements when appropriate
- Reduce unnecessary vector complexity

### Important Note

AI cannot properly read some built-in Figma effects such as Glass effects.

---

## Section Organization

Group layouts clearly by section, such as:

- Header
- Hero Section
- Introductory Content
- Gallery
- Footer

AI tools interpret layouts more effectively when sections are clearly grouped.

---

## Layer Naming

- Properly rename layers whenever possible
- If time is limited, prioritize naming top-level groups and sections accurately

Clear naming significantly improves developer readability and AI interpretation.

---

## Auto Layout Considerations

While Auto Layout is generally encouraged, some layouts may perform better when grouped manually instead.

### Take Note

- AI often reads layouts horizontally and section-by-section
- Overly complex Auto Layout structures may produce inconsistent results

If unsure whether a layout should remain in Auto Layout or be grouped differently, coordinate with the development team.

---

## Components

Standard component workflows and variants may still be used.

AI tools are generally capable of detecting component variants correctly.

### However

- Avoid deeply nested component structures where possible
- Limit nesting to approximately 1–2 levels when feasible
- Simplify component hierarchies to improve AI readability and implementation accuracy
