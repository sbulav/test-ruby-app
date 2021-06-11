# Use latest ruby base image at the moment
FROM ruby:3.0.1-alpine

# Add curl for health probes
RUN apk add --no-cache curl=7.77.0-r0

# Create a non-root user to run the app and own app-specific files
RUN adduser -D app

# Switch to this user
USER app

# We'll install the app in this directory
WORKDIR /app

# Copy the code
COPY --chown=app app/http_server.rb /app

# Notify that we expose port 80
EXPOSE 80

# Launch the server
ENTRYPOINT ["ruby", "http_server.rb"]
