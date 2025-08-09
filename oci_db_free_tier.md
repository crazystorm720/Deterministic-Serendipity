## **PostgreSQL Hosting: OCI Always Free + Home Lab Hybrid**

**Primary Setup: OCI Always Free Tier**
- Deploy PostgreSQL on the Arm-based Ampere A1 (4 cores, 24GB RAM)
- This gives you a production-grade, always-available database
- 200GB block storage is plenty for grants data
- 10TB monthly data transfer handles all your API calls and client deliveries

**Backup/Development: Home Lab**
- Keep a local PostgreSQL instance for development and testing
- Use it for experimentation without affecting production
- Disaster recovery backup location

**Why This Hybrid Approach:**
- **Cost**: Truly free forever (not 3-month AWS trial)
- **Performance**: 24GB RAM handles massive datasets easily
- **Reliability**: Cloud uptime for client-facing services
- **Security**: Your production data isn't on your home network
- **Scalability**: Room to grow without cost jumps

## **Database Schema for Grants Intelligence**

### **Core Tables Structure**

```sql
-- Raw grants data from Grants.gov
CREATE TABLE raw_grants (
    opportunity_id VARCHAR(50) PRIMARY KEY,
    opportunity_number VARCHAR(100),
    opportunity_title TEXT,
    cfda_number VARCHAR(20),
    cfda_title TEXT,
    funding_agency_code VARCHAR(10),
    funding_agency_name VARCHAR(200),
    award_ceiling DECIMAL(15,2),
    award_floor DECIMAL(15,2),
    posted_date DATE,
    close_date DATE,
    last_updated_date TIMESTAMP,
    opportunity_category VARCHAR(50),
    funding_instrument_type VARCHAR(100),
    eligible_applicants TEXT[],
    description TEXT,
    version_nb INTEGER,
    raw_xml_data JSONB  -- Store original XML for reference
);

-- Processed grants with intelligence flags
CREATE TABLE processed_grants (
    id SERIAL PRIMARY KEY,
    opportunity_id VARCHAR(50) REFERENCES raw_grants(opportunity_id),
    target_region VARCHAR(20), -- TX, OK, AR, etc.
    sector_category VARCHAR(100), -- Health, Education, etc.
    size_category VARCHAR(20), -- Small (50K-250K), Medium (250K-1M), Large (1M+)
    competition_level VARCHAR(20), -- Low, Medium, High (based on historical data)
    win_probability_score DECIMAL(3,2), -- 0.00 to 1.00
    strategic_notes TEXT,
    client_matches TEXT[], -- Array of client IDs this matches
    created_at TIMESTAMP DEFAULT NOW()
);

-- Historical award winners from USAspending.gov
CREATE TABLE award_winners (
    award_id VARCHAR(100) PRIMARY KEY,
    cfda_number VARCHAR(20),
    recipient_name VARCHAR(300),
    recipient_state VARCHAR(2),
    recipient_zip VARCHAR(10),
    award_amount DECIMAL(15,2),
    award_date DATE,
    funding_agency_name VARCHAR(200),
    project_title TEXT,
    project_description TEXT
);

-- Client profiles and preferences
CREATE TABLE clients (
    client_id SERIAL PRIMARY KEY,
    organization_name VARCHAR(200),
    contact_email VARCHAR(100),
    target_regions VARCHAR(10)[],
    focus_areas VARCHAR(100)[],
    min_award_size DECIMAL(15,2),
    max_award_size DECIMAL(15,2),
    cfda_preferences VARCHAR(20)[],
    subscription_tier VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Weekly intelligence reports
CREATE TABLE intelligence_reports (
    report_id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(client_id),
    report_date DATE,
    opportunities_found INTEGER,
    key_insights TEXT,
    report_data JSONB,
    delivered_at TIMESTAMP
);
```

### **Key Indexes for Performance**

```sql
-- Fast filtering for common queries
CREATE INDEX idx_grants_cfda ON raw_grants(cfda_number);
CREATE INDEX idx_grants_agency ON raw_grants(funding_agency_code);
CREATE INDEX idx_grants_dates ON raw_grants(posted_date, close_date);
CREATE INDEX idx_grants_amount ON raw_grants(award_ceiling, award_floor);

-- Geographic and sector filtering
CREATE INDEX idx_winners_state ON award_winners(recipient_state);
CREATE INDEX idx_winners_cfda ON award_winners(cfda_number);
CREATE INDEX idx_winners_amount ON award_winners(award_amount);

-- Client matching
CREATE INDEX idx_processed_matches ON processed_grants USING GIN(client_matches);
```

### **Intelligence Views for Quick Analysis**

```sql
-- Texas health grants under $2M
CREATE VIEW tx_health_opportunities AS
SELECT rg.*, pg.win_probability_score, pg.competition_level
FROM raw_grants rg
JOIN processed_grants pg ON rg.opportunity_id = pg.opportunity_id
WHERE rg.cfda_number LIKE '93.%'  -- Health & Human Services
  AND rg.award_ceiling <= 2000000
  AND pg.target_region = 'TX'
  AND rg.close_date > CURRENT_DATE;

-- Weekly trend analysis
CREATE VIEW weekly_funding_trends AS
SELECT 
    funding_agency_name,
    COUNT(*) as new_opportunities,
    AVG(award_ceiling) as avg_award_size,
    SUM(award_ceiling) as total_funding
FROM raw_grants 
WHERE posted_date >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY funding_agency_name
ORDER BY total_funding DESC;
```

## **Data Pipeline Architecture**

### **Weekly ETL Process**
1. **Extract**: Download Grants.gov XML files
2. **Transform**: Parse XML, clean data, calculate intelligence scores
3. **Load**: Insert into PostgreSQL with conflict resolution
4. **Analyze**: Update processed_grants with new insights
5. **Report**: Generate client-specific intelligence reports

### **Client Intelligence Queries**
```sql
-- Find grants matching a specific client profile
SELECT rg.opportunity_title, rg.funding_agency_name, 
       rg.award_ceiling, rg.close_date, pg.win_probability_score
FROM raw_grants rg
JOIN processed_grants pg ON rg.opportunity_id = pg.opportunity_id
WHERE pg.client_matches @> ARRAY['client_123']
  AND rg.close_date > CURRENT_DATE + INTERVAL '30 days'
ORDER BY pg.win_probability_score DESC;
```

This schema gives you:
- **Scalability**: Handle millions of grant records
- **Flexibility**: Easy to add new data sources (SAM.gov, etc.)
- **Performance**: Fast queries for client reports
- **Intelligence**: Built-in scoring and matching systems
- **Growth**: Room to add more sophisticated analytics

The OCI setup gives you enterprise-grade infrastructure at zero cost, while the schema design supports both your immediate needs and future expansion into contracts, more sophisticated AI analysis, and larger client bases.
