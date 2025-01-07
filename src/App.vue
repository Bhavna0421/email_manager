<template>
  <div class="email-manager">
    <h2>Email Manager</h2>

    <div class="email-add">
      <input
        type="text"
        v-model="newEmail"
        placeholder="Enter a new email"
        @keyup.enter="addNewEmail"
      />
      <button @click="addNewEmail">Add Email</button>
      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    </div>

    <div class="lists">
      <div class="available-list">
        <h3>Available Recipients</h3>
        <input 
          type="text" 
          v-model="searchInput" 
          placeholder="Enter company name or email" 
          @input="filterRecipients"
        />
        <ul>
          <li v-for="recipient in filteredRecipients" :key="recipient.email">
            <label>
              <input 
                type="checkbox" 
                :checked="recipient.isSelected" 
                @change="toggleRecipientSelection(recipient)" 
              />
              {{ recipient.email }}
            </label>
          </li>
        </ul>
      </div>

      <div class="selected-list">
        <h3>Selected Recipients</h3>
        <ul>
          <li v-for="(emails, domain) in groupedSelectedRecipients" :key="domain">
            <div>
              <strong>{{ domain }}</strong>
              <button @click="removeDomain(domain)">Remove All</button>
            </div>
            <ul>
              <li v-for="email in emails" :key="email">
                {{ email }}
                <button @click="removeEmail(email)">Remove</button>
              </li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      recipients: [
        { email: "ann@timescale.com", isSelected: false },
        { email: "bob@timescale.com", isSelected: false },
        { email: "brian@qwerty.com", isSelected: true },
        { email: "james@qwerty.com", isSelected: false },
        { email: "jane@awesome.com", isSelected: false },
        { email: "kate@qwerty.com", isSelected: true },
        { email: "mike@hello.com", isSelected: true }
      ],
      searchInput: "",
      filteredRecipients: [],
      newEmail: "",
      errorMessage: "",
    };
  },
  computed: {
    groupedSelectedRecipients() {
      return this.recipients
        .filter(r => r.isSelected)
        .reduce((acc, recipient) => {
          const domain = recipient.email.split("@")[1];
          if (!acc[domain]) acc[domain] = [];
          acc[domain].push(recipient.email);
          return acc;
        }, {});
    }
  },
  methods: {
    filterRecipients() {
      const query = this.searchInput.toLowerCase();
      this.filteredRecipients = this.recipients.filter(r => {
        return (
          r.email.toLowerCase().includes(query) ||
          r.email.split("@")[1].toLowerCase().includes(query)
        );
      });
    },
    toggleRecipientSelection(recipient) {
      recipient.isSelected = !recipient.isSelected;
    },
    removeEmail(email) {
      const recipient = this.recipients.find(r => r.email === email);
      if (recipient) recipient.isSelected = false;
    },
    removeDomain(domain) {
      this.recipients.forEach(r => {
        if (r.email.split("@")[1] === domain) {
          r.isSelected = false;
        }
      });
    },
    addNewEmail() {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

      if (!emailRegex.test(this.newEmail)) {
        this.errorMessage = "Please enter a valid email address.";
        return;
      }

      if (this.recipients.some(r => r.email === this.newEmail)) {
        this.errorMessage = "This email is already in the list.";
        return;
      }

      this.recipients.push({ email: this.newEmail, isSelected: false });
      this.newEmail = "";
      this.errorMessage = "";
      this.filterRecipients();
    },
  },
  created() {
    this.filteredRecipients = this.recipients;
  }
};
</script>

<style scoped>
.email-manager {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.email-add {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.error {
  color: red;
  font-size: 0.9rem;
}

.lists {
  display: flex;
  justify-content: space-between;
  gap: 2rem;
}

.available-list, .selected-list {
  width: 45%;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin: 0.5rem 0;
}
</style>
