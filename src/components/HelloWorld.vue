<template>
  <div class="hello">
    <textarea ref="input" autofocus></textarea>
  </div>
</template>

<script>
import Tagify from "@yaireo/tagify";

export default {
  name: "HelloWorld",
  props: {},
  methods: {
    validateEmail(email) {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    },
    getAttributes(tagData) {
      const attributes = [];
      for (const [key, value] of Object.entries(tagData)) {
        if (key !== "value" && key !== "email" && key !== "class") {
          attributes.push(`${key}="${value}"`);
        }
      }
      return attributes.join(" ");
    },

    parseFullValue(value) {
      // https://stackoverflow.com/a/11592042/104380
      var parts = value.split(/<(.*?)>/g),
        name = parts[0].trim(),
        email = parts[1]?.replace(/<(.*?)>/g, "").trim();

      return { name, email };
    },
    tagTemplate(tagData) {
      return `
        <tag title="${tagData.email}"
                contenteditable='false'
                spellcheck='false'
                tabIndex="-1"
                class="tagify__tag ${tagData.class ? tagData.class : ""}"
                ${this.getAttributes(tagData)}>
            <x title='' class='tagify__tag__removeBtn' role='button' aria-label='remove tag'></x>
            <div>
                <div class='tagify__tag__avatar-wrap'>
                    <img onerror="this.style.visibility='hidden'" src="${
                      tagData.avatar
                    }">
                </div>
                <span class='tagify__tag-text'>${tagData.name}</span>
            </div>
        </tag>
    `;
    },
    suggestionItemTemplate(tagData) {
      return `
        <div ${this.getAttributes(tagData)}
            class='tagify__dropdown__item ${tagData.class ? tagData.class : ""}'
            tabindex="0"
            role="option">
            ${
              tagData.avatar
                ? `
                <div class='tagify__dropdown__item__avatar-wrap'>
                    <img onerror="this.style.visibility='hidden'" src="${tagData.avatar}">
                </div>`
                : ""
            }
            <strong>${tagData.name}</strong>
            <span>${tagData.email}</span>
        </div>
    `;
    },

    dropdownHeaderTemplate(suggestions) {
      return `
        <header data-selector='tagify-suggestions-header' class="${
          this.settings.classNames.dropdownItem
        } ${this.settings.classNames.dropdownItem}__addAll">
            <div>
                <strong>${
                  this.value.length ? `Add Remaning` : "Add All"
                }</strong>
                <a class='remove-all-tags'>Remove all</a>
            </div>
            <span>${suggestions.length} members</span>
        </header>
    `;
    },
  },
  mounted() {
    const whitelist = [
      {
        value: 1,
        name: "Justinian Hattersley",
        avatar: "https://i.pravatar.cc/80?img=1",
        email: "jhattersley0@ucsd.edu",
        team: "A",
      },
      {
        value: 2,
        name: "Antons Esson",
        avatar: "https://i.pravatar.cc/80?img=2",
        email: "aesson1@ning.com",
        team: "B",
      },
      {
        value: 3,
        name: "Ardeen Batisse",
        avatar: "https://i.pravatar.cc/80?img=3",
        email: "abatisse2@nih.gov",
        team: "A",
      },
      {
        value: 4,
        name: "Graeme Yellowley",
        avatar: "https://i.pravatar.cc/80?img=4",
        email: "gyellowley3@behance.net",
        team: "C",
      },
      {
        value: 5,
        name: "Dido Wilford",
        avatar: "https://i.pravatar.cc/80?img=5",
        email: "dwilford4@jugem.jp",
        team: "A",
      },
      {
        value: 6,
        name: "Celesta Orwin",
        avatar: "https://i.pravatar.cc/80?img=6",
        email: "corwin5@meetup.com",
        team: "C",
      },
      {
        value: 7,
        name: "Sally Main",
        avatar: "https://i.pravatar.cc/80?img=7",
        email: "smain6@techcrunch.com",
        team: "A",
      },
      {
        value: 8,
        name: "Grethel Haysman",
        avatar: "https://i.pravatar.cc/80?img=8",
        email: "ghaysman7@mashable.com",
        team: "B",
      },
      {
        value: 9,
        name: "Marvin Mandrake",
        avatar: "https://i.pravatar.cc/80?img=9",
        email: "mmandrake8@sourceforge.net",
        team: "B",
      },
      {
        value: 10,
        name: "Corrie Tidey",
        avatar: "https://i.pravatar.cc/80?img=10",
        email: "ctidey9@youtube.com",
        team: "A",
      },
      {
        value: 11,
        name: "foo",
        avatar: "https://i.pravatar.cc/80?img=11",
        email: "foo@bar.com",
        team: "B",
      },
      {
        value: 12,
        name: "foo",
        avatar: "https://i.pravatar.cc/80?img=12",
        email: "foo.aaa@foo.com",
        team: "A",
      },
    ]; // initialize Tagify
    const input = this.$refs.input;
    // init Tagify script on the above inputs
    const tagify = new Tagify(input, {
      tagTextProp: "name", // very important since a custom template is used with this property as text
      // enforceWhitelist: true,
      skipInvalid: true, // do not remporarily add invalid tags
      dropdown: {
        closeOnSelect: false,
        enabled: 0,
        classname: "users-list",
        searchKeys: ["name", "email"], // very important to set by which keys to search for suggesttions when typing
      },
      templates: {
        tag: this.tagTemplate,
        dropdownItem: this.suggestionItemTemplate,
        dropdownHeader: this.dropdownHeaderTemplate,
      },
      whitelist: whitelist,
      transformTag: (tagData, originalData) => {
        var { name, email } = this.parseFullValue(tagData.name);
        tagData.name = name;
        tagData.email = email || tagData.email;
      },

      validate({ name, email }) {
        // when editing a tag, there will only be the "name" property which contains name + email (see 'transformTag' above)
        if (!email && name) {
          var parsed = this.parseFullValue(name);
          name = parsed.name;
          email = parsed.email;
        }

        if (!name) return "Missing name";
        if (!this.validateEmail(email)) return "Invalid email";

        return true;
      },
    });
    function onSelectSuggestion(e) {
      console.log("e.detail.event", e.detail);
      if (e.detail.event.target.matches(".remove-all-tags")) {
        tagify.removeAllTags();
      }

      // custom class from "dropdownHeaderTemplate"
      else if (
        e.detail.elm.classList.contains(
          `${tagify.settings.classNames.dropdownItem}__addAll`
        )
      )
        tagify.dropdown.selectAll();
    }

    function onEditStart({ detail: { tag, data } }) {
      tagify.setTagTextNode(tag, `${data.name} <${data.email}>`);
    }
    tagify
      .on("dropdown:select", onSelectSuggestion) // allows selecting all the suggested (whitelist) items
      .on("edit:start", onEditStart);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
@import "@yaireo/tagify/dist/tagify.css";
</style>
