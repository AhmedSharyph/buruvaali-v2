/**
 * Buruvaali v3 - Lightweight Custom Select Dropdown
 * No dependencies. Supports single, multiple, creatable.
 * https://github.com/yourusername/buruvaali
 */

class Buruvaali {
  constructor(selectEl) {
    this.selectEl = selectEl;
    this.multiple = !!selectEl.multiple;
    this.creatable = selectEl.classList.contains("creatable");
    this.placeholder = selectEl.querySelector('option[value=""]')?.text || "Select...";
    
    this.options = Array.from(selectEl.options)
      .filter(opt => opt.value !== "")
      .map(opt => ({ label: opt.text, value: opt.value }));

    this.selected = [];
    this.currentIndex = -1;

    this.root = document.createElement("div");
    this.root.className = "buruvaali";

    this.selection = document.createElement("div");
    this.selection.className = "buruvaali-selection";

    this.display = document.createElement("div");
    this.display.className = "buruvaali-display";
    this.display.textContent = this.placeholder;

    this.input = document.createElement("input");
    this.input.className = "buruvaali-search";
    this.input.autocomplete = "off";
    this.input.spellcheck = false;

    this.selection.append(this.display, this.input);
    this.root.append(this.selection);

    this.dropdown = document.createElement("div");
    this.dropdown.className = "buruvaali-dropdown";
    this.root.append(this.dropdown);

    // Hide original <select>
    this.selectEl.style.display = "none";
    this.selectEl.parentNode.insertBefore(this.root, this.selectEl.nextSibling);

    // Sync initial selected values
    this.syncFromSelect();

    // Setup events
    this.bindEvents();

    // Initial render
    this.filter("");
  }

  bindEvents() {
    this.selection.addEventListener("click", () => this.open());
    this.input.addEventListener("input", () => this.filter(this.input.value));
    this.input.addEventListener("keydown", (e) => this.handleKeys(e));
    document.addEventListener("click", (e) => {
      if (!this.root.contains(e.target)) this.close();
    });
  }

  filter(term) {
    const t = term.toLowerCase().trim();
    this.filteredOptions = this.options.filter(opt => 
      opt.label.toLowerCase().includes(t)
    );
    this.renderOptions();
    this.currentIndex = -1;
  }

  renderOptions() {
    this.dropdown.innerHTML = "";

    // No results + creatable → show "Create"
    if (this.filteredOptions.length === 0 && this.creatable && this.input.value.trim()) {
      const createEl = document.createElement("div");
      createEl.className = "buruvaali-option";
      createEl.style.fontStyle = "italic";
      createEl.textContent = `+ Create "${this.input.value}"`;
      createEl.addEventListener("click", () => this.createAndSelect(this.input.value));
      this.dropdown.appendChild(createEl);
    } 
    // No results at all
    else if (this.filteredOptions.length === 0) {
      const noEl = document.createElement("div");
      noEl.className = "buruvaali-option";
      noEl.style.color = "#999";
      noEl.style.cursor = "default";
      noEl.textContent = "No options found";
      this.dropdown.appendChild(noEl);
    } 
    // Show filtered options
    else {
      this.filteredOptions.forEach(opt => {
        const el = document.createElement("div");
        el.className = "buruvaali-option";
        if (this.selected.some(s => s.value === opt.value)) el.classList.add("active");

        // Highlight search match
        if (this.input.value.trim()) {
          const regex = new RegExp(`(${this.input.value.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi');
          el.innerHTML = opt.label.replace(regex, '<mark>$1</mark>');
        } else {
          el.textContent = opt.label;
        }

        el.addEventListener("click", () => this.select(opt));
        this.dropdown.appendChild(el);
      });
    }

    this.optionEls = Array.from(this.dropdown.children);
  }

  select(opt) {
    if (this.multiple) {
      if (!this.selected.some(s => s.value === opt.value)) {
        this.selected.push(opt);
      }
    } else {
      this.selected = [opt];
      this.close();
    }
    this.updateDisplay();
    this.syncToSelect();
  }

  createAndSelect(label) {
    const value = label.toLowerCase().replace(/\s+/g, "_");
    const newOpt = { label, value };
    if (!this.options.some(o => o.value === value)) {
      this.options.push(newOpt);
    }
    this.selected.push(newOpt);
    this.updateDisplay();
    this.syncToSelect();
    this.close();
  }

  updateDisplay() {
    this.selection.querySelectorAll(".buruvaali-tag").forEach(t => t.remove());

    if (this.selected.length === 0) {
      this.display.textContent = this.placeholder;
      this.display.style.color = "#999";
    } else if (this.multiple) {
      this.display.textContent = "";
      this.selected.forEach(opt => {
        const tag = document.createElement("div");
        tag.className = "buruvaali-tag";
        tag.textContent = opt.label;
        const closeBtn = document.createElement("span");
        closeBtn.textContent = "×";
        closeBtn.addEventListener("click", (e) => {
          e.stopPropagation();
          this.selected = this.selected.filter(s => s.value !== opt.value);
          this.updateDisplay();
          this.syncToSelect();
        });
        tag.appendChild(closeBtn);
        this.selection.insertBefore(tag, this.input);
      });
    } else {
      this.display.textContent = this.selected[0].label;
      this.display.style.color = "#000";
    }
  }

  syncToSelect() {
    this.selectEl.querySelectorAll("option").forEach(opt => opt.selected = false);
    this.selected.forEach(sel => {
      let opt = this.selectEl.querySelector(`option[value="${sel.value}"]`);
      if (!opt) {
        opt = document.createElement("option");
        opt.value = sel.value;
        opt.textContent = sel.label;
        this.selectEl.appendChild(opt);
      }
      opt.selected = true;
    });
  }

  syncFromSelect() {
    this.selected = Array.from(this.selectEl.selectedOptions).map(o => ({
      label: o.text,
      value: o.value
    }));
    this.updateDisplay();
  }

  open() {
    this.dropdown.style.display = "block";
    this.input.focus();
    this.filter(this.input.value);
  }

  close() {
    this.dropdown.style.display = "none";
    this.currentIndex = -1;
    this.input.value = "";
    this.filter("");
  }

  handleKeys(e) {
    if (e.key === "ArrowDown") {
      e.preventDefault();
      this.moveIndex(1);
    } else if (e.key === "ArrowUp") {
      e.preventDefault();
      this.moveIndex(-1);
    } else if (e.key === "Enter") {
      e.preventDefault();
      const active = this.optionEls?.[this.currentIndex];
      if (!active) return;
      if (active.textContent.startsWith("+ Create")) {
        this.createAndSelect(this.input.value);
      } else {
        const opt = this.filteredOptions[this.currentIndex];
        if (opt) this.select(opt);
      }
    } else if (e.key === "Backspace" && this.multiple && !this.input.value && this.selected.length > 0) {
      e.preventDefault();
      this.selected.pop();
      this.updateDisplay();
      this.syncToSelect();
    }
  }

  moveIndex(step) {
    if (!this.optionEls || this.optionEls.length === 0) return;
    this.currentIndex = (this.currentIndex + step + this.optionEls.length) % this.optionEls.length;
    this.optionEls.forEach(el => el.classList.remove("active"));
    this.optionEls[this.currentIndex].classList.add("active");
    this.optionEls[this.currentIndex].scrollIntoView({ block: "nearest" });
  }
}

// Auto-initialize all .buruvaali-select elements on DOMContentLoaded
document.addEventListener("DOMContentLoaded", () => {
  document.querySelectorAll(".buruvaali-select").forEach(select => {
    new Buruvaali(select);
  });
});
