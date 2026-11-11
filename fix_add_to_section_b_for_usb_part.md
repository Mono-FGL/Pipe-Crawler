    // Log data to file safely
    if (f != NULL) {
      fprintf(f, "%.4f, %.4f\n", pos_x, pos_y);
      
      // CRITICAL: Empty the RAM buffer and force an immediate physical save
      fflush(f); 
      fsync(fileno(f)); 
    }
